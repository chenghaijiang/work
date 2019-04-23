1、自己动手手写一个Mini版本的ORM框架。

```java
private static List<Member> select(String sql) {
    List<Member> result = new ArrayList<>();
    Connection con = null;
    PreparedStatement pstm = null;
    ResultSet rs = null;
    try {
        //1、加载驱动类
        Class.forName("com.mysql.jdbc.Driver");
        //2、建立连接
        con =
        DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/gp-vip-spring-db-demo","root","123456");
        //3、创建语句集
        pstm = con.prepareStatement(sql);
        //4、执行语句集
        rs = pstm.executeQuery();
        while (rs.next()){
        咕泡出品，必属精品 www.gupaoedu.com
        3
        Member instance = mapperRow(rs,rs.getRow());
        result.add(instance);
        }
    //5、获取结果集
    }catch (Exception e){
    	e.printStackTrace();
    }
    //6、关闭结果集、关闭语句集、关闭连接
    finally {
        try {
            rs.close();
            pstm.close();
            con.close();
        }catch (Exception e){
        	e.printStackTrace();
        }
    }
    return result;
}
private static Member mapperRow(ResultSet rs, int i) throws Exception {
    Member instance = new Member();
    instance.setId(rs.getLong("id"));
    instance.setName(rs.getString("name"));
    instance.setAge(rs.getInt("age"));
    instance.setAddr(rs.getString("addr"));
    return instance；
}
```

2、理解ORM框架的顶层设计原理

将非功能性代码和业务代码分离，先将resultset封装数据的代码逻辑分离，增加一个mapperRow方法，专门处理对结果的封装，利用反射机制，读取class信息和annotation信息，将数据库表中的列和类中的字段进行关联映射并赋值，以减少重复代码