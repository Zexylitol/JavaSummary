# 线程状态

## 操作系统层面

创建、就绪、运行、阻塞、终止

<center><img src="https://i.loli.net/2021/03/15/IdDkZchR3abEAwG.png"/></center>

<center><img src="https://i.loli.net/2021/03/15/91IO7KjJAfxTvWm.png"/></center>

## Java API层面

根据`Thread.State`枚举，分为六种状态：`NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED`

<center><img src="https://i.loli.net/2021/03/15/MNv9jDeX8oTuf2U.png"/></center>

<center><img src="https://i.loli.net/2021/03/15/5IaBmTye9ncS47E.png"/></center>

# 状态转换

假设有线程`t`:

## CASE1：`NEW -> RUNNABLE`

<center><img src="https://i.loli.net/2021/03/15/lMX8rtSgvYIeT1A.png"/></center>

## CASE2：`RUNNABLE <-> WAITING`

<center><img src="https://i.loli.net/2021/03/15/ZTtsVS9QCza6gF2.png"/></center>

## CASE3：`RUNNABLE <-> WAITING`

<center><img src="https://i.loli.net/2021/03/15/JXCD7R53NjMLHuw.png"/></center>

## CASE4：`RUNNABLE <-> WAITING`

<center><img src="https://i.loli.net/2021/03/15/TQPyvpicHMuBmCx.png"/></center>

## CASE5：`RUNNABLE <-> TIMED_WAITING`

<center><img src="https://i.loli.net/2021/03/15/rcYiLWHD8fpoGdy.png"/></center>

## CASE6：`RUNNABLE <-> TIMED_WAITING`

<center><img src="https://i.loli.net/2021/03/15/wFSlRLmsHk2ri16.png"/></center>

## CASE7：`RUNNABLE <-> TIMED_WAITING`

<center><img src="https://i.loli.net/2021/03/15/rkGmSpU6joNsIFA.png"/></center>

## CASE8：`RUNNABLE <-> TIMED_WAITING`

<center><img src="https://i.loli.net/2021/03/15/JWPtoYODIXhruyB.png"/></center>

## CASE9：`RUNNABLE <-> BLOCKED`

<center><img src="https://i.loli.net/2021/03/15/a9RFKJcwYz4OMm7.png"/></center>

## CASE10：`RUNNABLE <-> TERMINATED `

<center><img src="https://i.loli.net/2021/03/15/xkWYbC5AptnyZzq.png"/></center>

# 总结

<center><img src="https://img-blog.csdnimg.cn/20190728103754686.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzQyNDgzMzQx,size_16,color_FFFFFF,t_70"/></center>

