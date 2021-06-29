<!-- GFM-TOC -->

- [1 场景](#_1-场景)
- [2 处理方式](#_2-处理方式)
  - [2.1 悲观锁](#_21-悲观锁)
  - [2.2 乐观锁](#_22-乐观锁)
  - [2.3 一些特殊场景的解决方案](#_23-一些特殊场景的解决方案)
- [3 讨论](#_3-讨论)
  - [3.1 方案对比](#_31-方案对比)
  - [3.2 思考](#_32-思考)

<!-- GFM-TOC -->

# 1 场景

电商下单过程一般会涉及到两条SQL操作，查询和更新操作。正是两条SQL操作的存在，会导致一些问题。比如现在需要下单购买3个产品，查询出来还剩3个库存，如果以此为条件判断可以购买，继而去更新库存表是错误的。比如在查询到更新时间间隙内，另一个事务下单成功锁住了2个库存，此时库存为1。而如果需要下单购买3个产品的事务继续去更新表，就会出现库存为$1-3=-2$的情况，出现了超卖现象。

现在单独讨论一下下单时库存的处理方式，如下：

```java
@Transactional
public StockUpdateResponse updateStockById(StockUpdateRequest req) {
    //参数处理
    select remain from trip_stock where id = #{id} and date = #{date};
    //lock为需要下单锁住的库存
    if (remain >= lock){
        update trip_stock set remain = remain - #{lock} where id = #{id} and date = #{date}
    } 
    //返回结果
}
```

事务只能保证数据一致性的问题，不能解决多个事务同时执行产生的超卖问题。比如20个事务，某个时刻发现$remain >= lock$条件都满足，都会执行`update`操作，导致`remain`小于0，**数据一致性是满足了，但是业务逻辑就出现问题了**。其实问题出现在这个地方，当发现 $remain >= lock$条件满足到执行更新操作这段时间内，可能有其他事务对记录进行了更新，**如果能保证从查询到更新这段时间内没有事务对该记录进行操作，那么这种方案是可以正确运转的**。但是，<span style="color:blue">在一个并发系统中，这是很难保证的，此类问题可以通过乐观锁或者悲观锁来解决</span>。

# 2 处理方式

## 2.1 悲观锁

悲观锁：每次拿数据的时候都会认为别人会去修改它，所以每次取拿数据的时候都会上锁，其他人想拿到数据得等到锁释放才行

```java
@Transactional
public StockUpdateResponse updateStockById(StockUpdateRequest req) {
    // 参数处理
    select remain from trip_stock where id = #{id} and date = #{date} for update;
    // lock为需要下单锁住的库存
    if (remain >= lock){
        update trip_stock set remain = remain - #{lock} where id = #{id} and date = #{date}
    } 
    // 返回结果
}

```

`select ...for update`会给记录加上排他锁，直到事务结束才会释放锁。当发现$remain >= lock$满足时，执行`update`操作，从`select`到`update`这段时间不会有其他事务能过对该记录进行修改。从而解决问题。

## 2.2 乐观锁

乐观锁：每次拿数据的时候认为不会有人修改它，直到真正去更新的时候才会去判断之前有没有人修改过，通常通过版本号来实现。思路如下，直到更新时的version和之前select出来的一致时才算成功。

```java
@Transactional
public void optLock() {
    int result = 0;
    while (result != 1) {
        int version = select version from test where ....;
        result = update test set version = version+ 1 where version = #{version}
    }
}

```

在下单锁库存操作中，可以如下实现：同样也是更新时判断remain是否和之前`select`出来的remain相等来实现。

```java
@Transactional
public StockUpdateResponse updateStockById(StockUpdateRequest req) {
    // 参数处理
    int result = 0;
    while (result != 1) {
        select remain from trip_stock where id = #{id} and date = #{date};
        //lock为需要下单锁住的库存
        if (remain >= lock){
            result = update trip_stock set remain = remain - #{lock} where id = #{id} and date = #{date} and remain = #{remain}
        } else {
            break;
        }
    }
    //返回结果
}

```

## 2.3 一些特殊场景的解决方案

其实，库存这种场景没有必要要求更新时的remain等于之前`select`出来的remain，只要更新表时这条记录的remain大于等于需要的锁住的数量lock即可。实现方案如下：一条SQL解决问题，把锁的问题交给数据库自己解决。

```java
@Transactional
public StockUpdateResponse updateStockById(StockUpdateRequest req) {
    //参数处理
    //lock为需要下单锁住的库存
    int result = update test set remain = remain - #{lock} where id = #{id} and date = #{date} and remain >= #{lock}
    //返回结果
}

```

# 3 讨论

## 3.1 方案对比

乐观锁适用于读多于写的情况，读不加锁，能够提高系统吞吐量。但是当写增多的时候，更新时where条件不满足的情况会增多，上层retry次数也会增多，反而可能降低性能，不如用悲观锁。第三种解决方案只适用于一些特殊场景，相比于乐观锁，能够降低retry次数，只要业务条件满足即可更新，相比于悲观锁，能够降低锁住记录的时间，提升系统吞吐量。

## 3.2 思考

如果在悲观锁方案中第一步select操作不是加的悲观锁，而是加的共享锁来锁住记录，既能防止其他事务更新也能提升系统并发性。

```java
@Transactional
public StockUpdateResponse updateStockById(StockUpdateRequest req) {
    //参数处理
    select remain from trip_stock where id = #{id} and date = #{date} lock in share mode;
    //lock为需要下单锁住的库存
    if (remain >= lock){
        update trip_stock set remain = remain - #{lock} where id = #{id} and date = #{date}
    } 
    //返回结果
}

```

应该是不能解决问题的。原因如下：

两个事务A和B，A和B都成功执行了`select ... lock in share mode`操作，并且$remain >= lock$条件满足，此时都会去执行更新操作。A执行update会要求获得记录上的X锁，由于记录上B加上了S锁，A会等待B释放S锁；B执行update会要求获得记录上的X锁，由于记录A加上了S锁，B会等待A释放S锁。相互等待，产生死锁。