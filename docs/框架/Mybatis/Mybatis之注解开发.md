1. 删除`UserMapper.xml`

2. 定义`UserMapper`接口

   ```java
   package com.dao;
   
   import com.pojo.User;
   import org.apache.ibatis.annotations.*;
   
   import java.util.List;
   
   public interface UserMapper {
       @Select("select * from user")
       List<User> getUser();
   
       //方法存在多个参数，所有的参数必须加@Param
       @Select("select * from user where id = #{id}")
       User getUserById(@Param("id") int id);
   
       @Insert("insert into user (id, name, pwd) values" +
               "(#{id},#{name},#{pwd})")
       int addUser(User user);
   
       @Update("update user set name=#{name}, pwd=#{pwd} " +
               "where id=#{id}")
       int updateUser(User user);
   
       @Delete("delete from user where id=#{id}")
       int deleteUser(@Param("id") int id);
   }
   ```

3. 在核心配置文件mybatis-config.xml中**绑定接口**

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   
       <environments default="development">
           <environment id="development">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;
                   useUnicode=true&amp;characterEncoding=UTF-8"/>
                   <property name="username" value="root"/>
                   <property name="password" value="yzz1997"/>
               </dataSource>
           </environment>
       </environments>
   
       <!--  绑定接口  -->
       <mappers>
           <mapper class="com.dao.UserMapper"></mapper>
       </mappers>
   
   </configuration>
   ```

4. 编写测试

   ```java
   package com.dao;
   
   import com.pojo.User;
   import com.utils.MybatisUtils;
   import org.apache.ibatis.session.SqlSession;
   import org.junit.Test;
   
   import java.util.List;
   
   public class UserMapperTest {
       @Test
       public void getUserTest() {
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           List<User> user = mapper.getUser();
           for (User u : user) {
               System.out.println(u);
           }
           sqlSession.close();
       }
   
       @Test
    public void getuserByIdTest(){
           // 获得sqlsession对象
        SqlSession sqlSession = MybatisUtils.getSqlSession();
           try{
               // 1.执行 getmapper
               UserMapper userDao = sqlSession.getMapper(UserMapper.class);
               User user = userDao.getUserById(1);
   
               System.out.println(user);
   
   
           }catch(Exception e){
               e.printStackTrace();
           }finally{
               //关闭
               sqlSession.close();
           }
       }
   
       @Test
       public void addUserTest(){
           // 获得sqlsession对象
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           try{
               // 1.执行 getmapper
               UserMapper userDao = sqlSession.getMapper(UserMapper.class);
               userDao.addUser(new User(6, "uestc","123"));
               // 2. 提交事务
               sqlSession.commit();
   
           }catch(Exception e){
               e.printStackTrace();
           }finally{
               //关闭
               sqlSession.close();
           }
       }
       @Test
       public void updateUserTest(){
           // 获得sqlsession对象
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           try{
               // 1.执行 getmapper
               UserMapper userDao = sqlSession.getMapper(UserMapper.class);
               userDao.updateUser(new User(6, "nuc","123"));
               // 2. 提交事务
               sqlSession.commit();
           }catch(Exception e){
               e.printStackTrace();
           }finally{
               //关闭
               sqlSession.close();
           }
       }
   
       @Test
       public void deleteUserTest(){
           // 获得sqlsession对象
           SqlSession sqlSession = MybatisUtils.getSqlSession();
           try{
               // 1.执行 getmapper
               UserMapper userDao = sqlSession.getMapper(UserMapper.class);
               userDao.deleteUser(6);
               // 2. 提交事务
               sqlSession.commit();
           }catch(Exception e){
               e.printStackTrace();
           }finally{
               //关闭
               sqlSession.close();
           }
       }
   }
   ```
   
   本质：反射机制
   
   底层：动态代理

