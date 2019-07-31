[reference1](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483745&idx=2&sn=87b3b5d3b8d44b89743e9e15d3c3a80c&chksm=ebd74060dca0c97616b24b36bd053704ff88b241f11bc0914f9ed4b97f32f346af283d7a04b1#rd)  [reference2](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483745&idx=3&sn=26cc7194bca0f460f900176156b1795d&chksm=ebd74060dca0c97674f0ff43d7d9d831f7c773d8e5675e5d488356f5f3eb14283bd3be0faaba#rd)   [reference4](https://mp.weixin.qq.com/s?__biz=MzI4Njg5MDA5NA==&mid=2247483745&idx=5&sn=a3010d60de8afe16bb1367ab45ab257e&chksm=ebd74060dca0c97677aa069b74a8427a63fd4ede563e36d6b8ddd0f3f5db1e43f79145100cce#rd)



## 1、JDBC基础



**JDBC全称为：Java Data Base Connectivity,它是可以执行SQL语句的Java API**

**为什么使用JDBC?**

- 市面上有非常多的数据库，本来我们是需要根据不同的数据库学习不同的API，sun公司为了简化这个操作，定义了JDBC API【接口】
- sun公司只是提供了JDBC API【接口】，数据库厂商负责实现。
- 对于我们来说，**操作数据库都是在JDBC API【接口】上**，使用不同的数据库，只要用数据库厂商提供的数据库驱动程序即可

**使用JDBC:**

步骤:

1. 导入MySQL或者Oracle驱动包
2. 装载数据库驱动程序
3. 获取到与数据库连接
4. 获取可以执行SQL语句的对象
5. 执行SQL语句
6. 关闭连接

```java
Connection connection = null;
Statement statement = null;
ResultSet resultSet = null;
try {
    /*
            * 加载驱动有两种方式
            *
            * 1：会导致驱动会注册两次，过度依赖于mysql的api，脱离的mysql的开发包，程序则无法编译
            * 2：驱动只会加载一次，不需要依赖具体的驱动，灵活性高
            *
            * 我们一般都是使用第二种方式
            * */
    //1.
    //DriverManager.registerDriver(new com.mysql.jdbc.Driver());
    //2.
    Class.forName("com.mysql.jdbc.Driver");
    //获取与数据库连接的对象-Connetcion
    connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/zhongfucheng", "root", "root");
    //获取执行sql语句的statement对象
    statement = connection.createStatement();
    //执行sql语句,拿到结果集
    resultSet = statement.executeQuery("SELECT * FROM users");
    //遍历结果集，得到数据
    while (resultSet.next()) {
        System.out.println(resultSet.getString(1));
        System.out.println(resultSet.getString(2));
    }
} catch (SQLException e) {
    e.printStackTrace();
} catch (ClassNotFoundException e) {
    e.printStackTrace();
} finally {
    /*
            * 关闭资源，后调用的先关闭
            *
            * 关闭之前，要判断对象是否存在
            * */
    if (resultSet != null) {
        try {
            resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (statement != null) {
        try {
            statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (connection != null) {
        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

**JDBC中常用对象：**

==**Connection**==: 

​	客户端与数据库所有的交互都是通过Connection来完成的

```java
//创建向数据库发送sql的statement对象。
createcreateStatement()
//创建向数据库发送预编译sql的PrepareSatement对象。
prepareStatement(sql) 
//创建执行存储过程的callableStatement对象
prepareCall(sql)
//设置事务自动提交
setAutoCommit(boolean autoCommit)
//提交事务
commit()
//回滚事务
rollback()
```

**==Statement==**:

​	Statement对象用于向数据库发送Sql语句，对数据库的增删改查都可以通过此对象发送sql语句完成。

```java
//查询
executeQuery(String sql)
//增删改
executeUpdate(String sql)
//任意sql语句都可以，但是目标不明确，很少用
execute(String sql)
//把多条的sql语句放进同一个批处理中
addBatch(String sql)
//向数据库发送一批sql语句执行
executeBatch()
```

**==ResultSet==**:

​	ResultSet对象代表Sql语句的执行结果，当Statement对象执行executeQuery()时，会返回一个ResultSet对象

​	ResultSet对象维护了一个数据行的游标【简单理解成指针】，调用ResultSet.next()方法，可以让游标指向具体的数据行，进行获取该行的数据

```java
//获取任意类型的数据
getObject(String columnName)
//获取指定类型的数据【各种类型，查看API】
getString(String columnName)
//对结果集进行滚动查看的方法
next()
Previous()
absolute(int row)
beforeFirst()
afterLast()
```

**简单的工具类：**

```java
/*
 * 连接数据库的driver，url，username，password通过配置文件来配置，可以增加灵活性
 * 当我们需要切换数据库的时候，只需要在配置文件中改以上的信息即可
 * */
private static String driver = null;
private static String url = null;
private static String username = null;
private static String password = null;
static {
    try {
        //获取配置文件的读入流
        InputStream inputStream = UtilsDemo.class.getClassLoader().getResourceAsStream("db.properties");
        Properties properties = new Properties();
        properties.load(inputStream);
        //获取配置文件的信息
        driver = properties.getProperty("driver");
        url = properties.getProperty("url");
        username = properties.getProperty("username");
        password = properties.getProperty("password");
        //加载驱动类
        Class.forName(driver);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    }
}
public static Connection getConnection() throws SQLException {
    return DriverManager.getConnection(url,username,password);
}
public static void release(Connection connection, Statement statement, ResultSet resultSet) {
    if (resultSet != null) {
        try {
            resultSet.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (statement != null) {
        try {
            statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    if (connection != null) {
        try {
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

## 2、批处理，大文本

**PrepareStatement**：

**PreparedStatement对象继承Statement对象，它比Statement对象更强大，使用起来更简单**

1. Statement对象编译SQL语句时，如果SQL语句有变量，就需要使用分隔符来隔开，如果变量非常多，就会使SQL变得非常复杂。**PreparedStatement可以使用占位符，简化sql的编写**
2. Statement会频繁编译SQL。**PreparedStatement可对SQL进行预编译，提高效率，预编译的SQL存储在PreparedStatement对象中**
3. **PreparedStatement防止SQL注入**。【Statement通过分隔符'++',编写永等式，可以不需要密码就进入数据库】

```java
//模拟查询id为2的信息
String id = "2";
Connetion connection = UtilsDemo.getConnection();
String sql = "SELECT * FROM users WHERE id = ?";
PreparedStatement preparedStatement = connection.preparedStatement(sql);
//第一个参数表示第几个占位符【也就是?号】，第二个参数表示值是多少
preparedStatement.setString(1, id);
ResultSet resultSet = preparedStatement.executeQuery();
if (resultSet.next()) {
    System.out.println(resultSet.getString("name"));
}
//释放资源
UtilsDemo.release(connection, preparedStatement, resultSet);
```

**批处理：**

当需要向数据库发送一批SQL语句执行时，**采用批处理以提升执行效率**

**通过executeBatch()方法批量处理执行SQL语句，返回一个int[]数组，该数组代表各句SQL的返回值**

Statement和PreparedStatement都可以进行批处理工作

```java
/*
 * Statement执行批处理
 *
 * 优点：
 *       可以向数据库发送不同的SQL语句
 * 缺点：
 *       SQL没有预编译
 *       仅参数不同的SQL，需要重复写多条SQL
 * */
Connection connection = UtilsDemo.getConnection();
Statement statement = connection.createStatement();
String sql1 = "UPDATE users SET name='zhongfucheng' WHERE id='3'";
String sql2 = "INSERT INTO users (id, name, password, email, birthday)" +
    " VALUES('5','nihao','123','ss@qq.com','1995-12-1')";
//将sql添加到批处理
statement.addBatch(sql1);
statement.addBatch(sql2);
//执行批处理
statement.executeBatch();
//清空批处理的sql
statement.clearBatch();
UtilsDemo.release(connection, statement, null);


/*
 * PreparedStatement批处理
 *   优点：
 *       SQL语句预编译了
 *       对于同一种类型的SQL语句，不用编写很多条
 *   缺点：
 *       不能发送不同类型的SQL语句
 *
 * */
Connection connection = UtilsDemo.getConnection();
String sql = "INSERT INTO test(id,name) VALUES (?,?)";
PreparedStatement preparedStatement = connection.prepareStatement(sql);
for (int i = 1; i <= 205; i++) {
    preparedStatement.setInt(1, i);
    preparedStatement.setString(2, (i + "zhongfucheng"));
    //添加到批处理中
    preparedStatement.addBatch();
    if (i %2 ==100) {
        //执行批处理
        preparedStatement.executeBatch();
        //清空批处理【如果数据量太大，所有数据存入批处理，内存肯定溢出】
        preparedStatement.clearBatch();
    }
}
//不是所有的%2==100，剩下的再执行一次批处理
preparedStatement.executeBatch();
//再清空
preparedStatement.clearBatch();
UtilsDemo.release(connection, preparedStatement, null);
```

**处理大文本和二进制数据**

**MySQL存储大文本是用Test**，Test分为：

- TINYTEXT
- TEXT
- MEDIUMTEXT
- LONGTEXT

```java
/*
*用JDBC操作MySQL数据库去操作大文本数据
*
*setCharacterStream(int parameterIndex,java.io.Reader reader,long length)
*第二个参数接收的是一个流对象，因为大文本不应该用String来接收，String太大会导致内存溢出
*第三个参数接收的是文件的大小
*
* */
public class Demo5 {
    @Test
    public void add() {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JdbcUtils.getConnection();
            String sql = "INSERT INTO test2 (bigTest) VALUES(?) ";
            preparedStatement = connection.prepareStatement(sql);
            //获取到文件的路径
            String path = Demo5.class.getClassLoader().getResource("BigTest").getPath();
            File file = new File(path);
            FileReader fileReader = new FileReader(file);
            //第三个参数，由于测试的Mysql版本过低，所以只能用int类型的。高版本的不需要进行强转
            preparedStatement.setCharacterStream(1, fileReader, (int) file.length());
            if (preparedStatement.executeUpdate() > 0) {
                System.out.println("插入成功");
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(connection, preparedStatement, null);
        }
    }
    /*
    * 读取大文本数据，通过ResultSet中的getCharacterStream()获取流对象数据
    * 
    * */
    @Test
    public void read() {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JdbcUtils.getConnection();
            String sql = "SELECT * FROM test2";
            preparedStatement = connection.prepareStatement(sql);
            resultSet = preparedStatement.executeQuery();
            if (resultSet.next()) {
                Reader reader = resultSet.getCharacterStream("bigTest");
                FileWriter fileWriter = new FileWriter("d:\\abc.txt");
                char[] chars = new char[1024];
                int len = 0;
                while ((len = reader.read(chars)) != -1) {
                    fileWriter.write(chars, 0, len);
                    fileWriter.flush();
                }
                fileWriter.close();
                reader.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(connection, preparedStatement, resultSet);
        }      
    }
```



**获取数据库的自动主键列：**

```java
@Test
public void test() {
    Connection connection = null;
    PreparedStatement preparedStatement = null;
    ResultSet resultSet = null;
    try {
        connection = JdbcUtils.getConnection();
        String sql = "INSERT INTO test(name) VALUES(?)";
        preparedStatement = connection.prepareStatement(sql);
        preparedStatement.setString(1, "ouzicheng");
        if (preparedStatement.executeUpdate() > 0) {
            //获取到自动主键列的值
            resultSet = preparedStatement.getGeneratedKeys();
            if (resultSet.next()) {
                int id = resultSet.getInt(1);
                System.out.println(id);
            }
        }       
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        JdbcUtils.release(connection, preparedStatement, null);
    }
```



## 4、数据库连接池、DbUtils框架、分页

**数据库连接池：**

提供连接。

- 数据库的连接的建立和关闭是非常消耗资源的
- 频繁地打开、关闭连接造成系统性能低下

**编写连接池：**

1. 编写连接池需**实现java.sql.DataSource接口**
2. **创建批量的Connection用LinkedList保存**
3. **实现getConnetion()**，让getConnection()每次调用，都是**在LinkedList中取一个Connection返回给用户**
4. **调用Connection.close()方法，Connction返回给LinkedList**

```java
private static LinkedList<Connection> list = new LinkedList<>();
//获取连接只需要一次就够了，所以用static代码块
static {
    //读取文件配置
    InputStream inputStream = Demo1.class.getClassLoader().getResourceAsStream("db.properties");
    Properties properties = new Properties();
    try {
        properties.load(inputStream);
        String url = properties.getProperty("url");
        String username = properties.getProperty("username");
        String driver = properties.getProperty("driver");
        String password = properties.getProperty("password");
        //加载驱动
        Class.forName(driver);
        //获取多个连接，保存在LinkedList集合中
        for (int i = 0; i < 10; i++) {
            Connection connection = DriverManager.getConnection(url, username, password);
            list.add(connection);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } catch (ClassNotFoundException e) {
        e.printStackTrace();
    } catch (SQLException e) {
        e.printStackTrace();
    }
}

//重写Connection方法，用户获取连接应该从LinkedList中给他
@Override
public Connection getConnection() throws SQLException {
    System.out.println(list.size());
    System.out.println(list);
    //先判断LinkedList是否存在连接
    return list.size() > 0 ? list.removeFirst() : null; 
}
```



当要关闭Connection的时候，**使用动态代理，返回代理对象，拦截close()方法**：

```java
@Override
public Connection getConnection() throws SQLException {
    if (list.size() > 0) {
        final Connection connection = list.removeFirst();
        //看看池的大小
        System.out.println(list.size());
        //返回一个动态代理对象
        return (Connection) Proxy.newProxyInstance(Demo1.class.getClassLoader(), connection.getClass().getInterfaces(), new InvocationHandler()){
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //如果不是调用close方法，就按照正常的来调用
                if (!method.getName().equals("close")) {
                    method.invoke(connection, args);
                } else {
                    //进到这里来，说明调用的是close方法
                    list.add(connection);
                    //再看看池的大小
                    System.out.println(list.size());
                }
                return null;
            }
        });
    }
    return null;
}
```



**DBCP:**

```java
private static DataSource dataSource = null;

static {
    try {
        //读取配置文件
        InputStream inputStream = Demo3.class.getClassLoader().getResourceAsStream("dbcpconfig.properties");
        Properties properties = new Properties();
        properties.load(inputStream);
        //获取工厂对象
        BasicDataSourceFactory basicDataSourceFactory = new BasicDataSourceFactory();
        dataSource = basicDataSourceFactory.createDataSource(properties);
    } catch (IOException e) {
        e.printStackTrace();
    } catch (Exception e) {
        e.printStackTrace();
    }
}

public static Connection getConnection() throws SQLException {
    return dataSource.getConnection();
}

//这里释放资源不是把数据库的物理连接释放了，是把连接归还给连接池【连接池的Connection内部自己做好了】
public static void release(Connection conn, Statement st, ResultSet rs) {
    if (rs != null) {
        try {
            rs.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        rs = null;
    }
    if (st != null) {
        try {
            st.close();
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
    if (conn != null) {
        try {
            conn.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



**C3P0：**

可以**使用XML配置文件配置信息**



**Tomcat数据源：**

步骤：

1. 在META-INF目录下配置context.xml文件【文件内容可以在tomcat默认页面的 JNDI Resources下Configure Tomcat's Resource Factory找到】
2. 导入Mysql或oracle开发包到tomcat的lib目录下
3. 初始化JNDI->获取JNDI容器->检索以XXX为名字在JNDI容器存放的连接池

context.xml文件的配置：

```xml
<Context>
  <Resource name="jdbc/EmployeeDB"
            auth="Container"
            type="javax.sql.DataSource"
            
            username="root"
            password="root"
            driverClassName="com.mysql.jdbc.Driver"
            url="jdbc:mysql://localhost:3306/zhongfucheng"
            maxActive="8"
            maxIdle="4"/>
</Context>
```

```java
try {
    //初始化JNDI容器
    Context initCtx = new InitialContext();
    //获取到JNDI容器
    Context envCtx = (Context) initCtx.lookup("java:comp/env");
    //扫描以jdbc/EmployeeDB名字绑定在JNDI容器下的连接池
    DataSource ds = (DataSource) envCtx.lookup("jdbc/EmployeeDB");
    Connection conn = ds.getConnection();
    System.out.println(conn);
} 
```



**dbutils框架**



**DbUtils类**

提供了**关闭连接，装载JDBC驱动，回滚提交事务等方法**的工具类



**QueryRunner类**

该类**简化SQL查询，配合ResultSetHandler使用，可以完成大部分的数据库操作**



**ResultSetHandler接口**

该接口**规范了对ResultSet的操作**，要对结果集进行什么操作，传入ResultSetHandler接口的实现类即可



**dbutils的CRUD**:

```java
/*
* 使用DbUtils框架对数据库的CRUD
* 批处理
*
* */
public class Test {
    @org.junit.Test
    public void add() throws SQLException {
        //创建出QueryRunner对象
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "INSERT INTO student (id,name) VALUES(?,?)";
        //我们发现query()方法有的需要传入Connection对象，有的不需要传入
        //区别：你传入Connection对象是需要你来销毁该Connection，你不传入，由程序帮你把Connection放回到连接池中
        queryRunner.update(sql, new Object[]{"100", "zhongfucheng"});
    }

    @org.junit.Test
    public void query() throws SQLException {
        //创建出QueryRunner对象
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "SELECT * FROM student";
        List list = (List) queryRunner.query(sql, new BeanListHandler(Student.class));
        System.out.println(list.size());
    }

    @org.junit.Test
    public void delete() throws SQLException {
        //创建出QueryRunner对象
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "DELETE FROM student WHERE id='100'";
        queryRunner.update(sql);
    }

    @org.junit.Test
    public void update() throws SQLException {
        //创建出QueryRunner对象
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "UPDATE student SET name=? WHERE id=?";
        queryRunner.update(sql, new Object[]{"zhongfuchengaaa", 1});
    }

    @org.junit.Test
    public void batch() throws SQLException {
        //创建出QueryRunner对象
        QueryRunner queryRunner = new QueryRunner(JdbcUtils.getDataSource());
        String sql = "INSERT INTO student (name,id) VALUES(?,?)";
        Object[][] objects = new Object[10][];
        for (int i = 0; i < 10; i++) {
            objects[i] = new Object[]{"aaa", i + 300};
        }
        queryRunner.batch(sql, objects);
    }
}
```

 List list = (List) queryRunner.query(sql, new BeanListHandler(Student.class));



