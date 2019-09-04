## 一、Java基础

### 1、map的分类和常见情况

map是键值对的集合接口，实现主要有四类：HashMap, TreeMap,Hashtable以及LinkedHashMap

- **HashMap**：根据key值得hashcode来存储数据，最多允许一条记录的key值为空，非同步
- **TreeMap**：对key值进行排序，用iterator遍历的时候是升序的，非同步
- **Hashtable**：与HashMap类似，key和value不允许为null；支持线程同步，同一时刻只能有一个线程能写Hashtable，因此速度较慢
- **LinkedHashMap**：保存了记录的插入顺序，key和value均允许为空，非同步

TreeMap的排序：

```java
//根据key值排序
public class TreeMapTest {
    public static void main(String[] args) {
        Map<String, String> map = new TreeMap<String, String>(
                new Comparator<String>() {
                    public int compare(String obj1, String obj2) {
                        // 降序排序
                        return obj2.compareTo(obj1);
                    }
                });
        map.put("c", "ccccc");
        map.put("a", "aaaaa");
        map.put("b", "bbbbb");
        map.put("d", "ddddd");

        Set<String> keySet = map.keySet();
        Iterator<String> iter = keySet.iterator();
        while (iter.hasNext()) {
            String key = iter.next();
            System.out.println(key + ":" + map.get(key));
        }
    }
}

//根据value值排序，借助Collection.sort（）
public class TreeMapTest {
    public static void main(String[] args) {
        Map<String, String> map = new TreeMap<String, String>();
        map.put("d", "ddddd");
        map.put("b", "bbbbb");
        map.put("a", "aaaaa");
        map.put("c", "ccccc");

        //这里将map.entrySet()转换成list
        List<Map.Entry<String,String>> list = new ArrayList<Map.Entry<String,String>>(map.entrySet());
        //然后通过比较器来实现排序
        Collections.sort(list,new Comparator<Map.Entry<String,String>>() {
            //升序排序
            public int compare(Entry<String, String> o1,
                    Entry<String, String> o2) {
                return o1.getValue().compareTo(o2.getValue());
            }

        });

        for(Map.Entry<String,String> mapping:list){ 
               System.out.println(mapping.getKey()+":"+mapping.getValue()); 
          } 
    }
}
```



### 2、Java8新特性

1. **Lambda表达式**：函数作为一个方法的参数，或者把代码看成数据

   ```java
   Arrays.asList( "p", "u","f", "o", "r","k").forEach( e -> System.out.println( e ) );
   ```

2. **方法引用**：与Lambda表达式配合，可以直接引用Java类或对象的方法

   - 构造器引用：语法是Class::new，要求构造器无参数
   - 静态方法引用：语法是Class::static_method，要求接受一个Class类型的参数
   - 特定类的任意方法引用：语法是Class::method。要求方法是没有参数的
   - 特定对象的方法引用：语法是instance::method

3. **接口的默认方法与静态方法**：在接口内部实现一个方法，用**default**关键字，所有实现这个接口的类接受这个默认方法，除非重写

   ```java
   public interface DefaultFunctionInterface {
       default String defaultFunction() {
           return "default function";
       }
   }
   
   public interface StaticFunctionInterface {
       static String defaultFunction() {
           return "static function";
       }
   }
   ```

### 3、 Lambda表达式优缺点

lambda: 将行为传递到函数中

- lambda语法

  ```java
  expression = (varialble) -> action
  
  BinaryOperator<Integer> sumFunction = (x, y) -> x + y;
  int sum = sumFunction.apply(1, 2);
  
  //实现Runnable
  new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
  
  //
  ```

- lambda可以说是一个 “ 函数式接口 ” 的实现

- lambda优点：

  - 配合FunctionalInterface Lib, forEach, stream()，method reference等新特性可以使代码变的更加简洁：

    例如：检查某一个对象并打印出，这里利用了JAVA内置接口Predicate 和 

    ```java
    //main
    
    public static void main(String[] args) {
            List<Person> lambdaTester = Arrays.asList(new Person("a", 15), new Person("b", 17));
            lambdaTester.stream().filter(person -> person.getName().equals("a")).forEach(System.out::println);
    }
    
    //Person
    public class Person {
        private String name;
        private int age;
        public Person(String name, int age){
            this.name = name;
            this.age = age;
        }
    
        public String getName() {
            return name;
        }
    
        public int getAge() {
            return age;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public void setAge(int age) {
            this.age = age;
        }
    
        @Override
        public String toString() {
            return "Person{" +
                    "name='" + name + '\'' +
                    ", age=" + age +
                    '}';
        }
    }
    
    //output
    Person{name='a', age=15}
    
    //一些解释:两个JAVA内置接口，可以作为stream方法的输入
    interface Predicate<T>{
        boolean test();
    }
    
    interface Consumer<T>{
        void accept();
    }
    
    stream().filter().forEach();
    
    //最后使用了Method referrence，用写好的别的Object/Class的method来代替Lambda expression
    Object::methodName
    Class::methodName
    例如: System.out::println;
    ```

  - 配合optional使用，处理null值

    ```java
    public static void main(String[] args) {
        Person a = new Person("a", 15);
        Optional<Person> personOpt = Optional.ofNullable(a);
        String str = personOpt.map(p -> p.getName()).map(p -> p.toUpperCase()).orElse(null);
        System.out.println(str);
    }
    
    return personOpt.orElse(null);
    personOpt.ifPresent(System.out.println);
    ```




### 4、十进制数是怎么表示的

用补码表示

- 补码：
  - 正数的补码与自身一致
  - 负数的反码除符号位以外，其他的各位取相反数，补码在反码基础上加1



### 5、为啥有时会出现4.0-3.6=0.40000001这种现象

[参考](https://www.cnblogs.com/yewsky/articles/1864934.html) 写得很详细

因为十进制数的二进制表示有可能不精确

- 小数的二进制表示：
  - 十进制整数转化为二进制数，除以2取模，直到商为0（商一定会为0，所以精确）
  - 十进制小数转化为整数，乘以2取整数部分，并用小数部分乘2继续，直到小数部分为0（小数部分可能不会为0，因此可能不精确）

- float型的存储方式（4Bytes）：

  ![1560132532280](C:\Users\xieyu\AppData\Roaming\Typora\typora-user-images\1560132532280.png)

  误差发生在存储时，截取多余循环的小数位然后放置在有效数位（共23位）中

  举例：11.9的存储（负数则保存绝对值的二进制，符号位设置）

  1.   11.9化为二进制1011.11100110011.....
  2. 小数点左移三位到第一个有效位，保证有效位数24位，1.01111101... 并截取多余的位数（产生误差）
  3. 去掉最左边的1，将剩下的23位放在22到0位
  4. 31位符号位置1
  5. 小数点左移，30位指数符号位放1
  6. 左移三位，则将3减1得到2，化为二进制，补足7位得到0000010放入第29-23位

  

### 6、Java支持的数据类型？自动拆装箱

[自动拆装箱](https://zhuanlan.zhihu.com/p/27657548)

- 基础数据类型：
  - 整型：byte, short, int, long
  - 布尔型：boolean
  - 浮点型：float, double
  - 字符型：char
- 引用类型



自动拆装箱：

装箱：valueOf

当变量声明为对象类型而赋值为基本数据类型时，Java编译器会对我们的基本数据类型进行装箱，而我们的变量声明为基本类型赋值为对象类型时，编译器又会对我们的对象类型进行拆箱处理，在编译成class文件中产生

- 为什么要有包装类：
  - 面向对象，基础数据类型需要初始化，而Integer可以直接设置为null，提供了额外的语义
  - 对泛型提供支持
  - 提供其他的属性以及相关的api



### 7、值传递与引用传递

[参考](https://www.zhihu.com/question/31203609/answer/576030121)

Java都是值传递，只是在传递引用对象的时候，将实参对象引用的地址复制了一份传递给形参

- 值传递与引用传递：

  - 值传递：调用函数时将实际参数复制一份（创建副本，无法改变原对象）
  - 引用传递：调用函数时将实际参数的地址传递到函数中（不创建副本，可以改变原对象）

  

  

### 8、Array([])与ArrayList的区别

[参考](https://www.cnblogs.com/wangbin2188/p/6524200.html)

- Array与ArrayList区别：

  - Array高效但是容量是固定的，ArrayList可动态增长但牺牲了效率
  - Array可以存储基础的数据类型（如int，byte，short，char...）ArrayList只能存储引用对象类型，并保存他们的引用地址

- Array的一些其他特点：

  - Array也是一个对象，一个Array类型的变量名表示一个堆上的引用地址，Array可以存储基础数据类型的值，也可以存储引用对象的地址
  - 由于Array表示一个引用地址，因此可作为函数的返回值
  - 基于效率或者类型检验，尽量使用Array

- ArrayList的一些特点：

  - ArrayList存储的是引用地址，这些引用地址指向Object对象
  - ArrayList低效的原因：
    - 添加元素时会检查内部空间是否足够
    - 存储对象时，会抛弃类型信息，将对象屏蔽为Object，编译不检查，运行时检查类型
    - 可以存任意的Object

- Arrays的常见操作:

  ```java
  Arrays.sort();
  Arrays.binarySearch();
  Arrays.equals();
  Arrays.asList();
  Arrays.fill();
  ```

  

### 9、String是基础类型吗？

不是

### 10、Integer和int的区别？

- Integer是int的包装类，int则是java的一种基本数据类型 

- Integer变量必须实例化后才能使用，而int变量不需要 

- Integer实际是对象的引用，当new一个Integer时，实际上是生成一个指针指向此对象；而int则是直接存储数据值 

- Integer的默认值是null，int的默认值是0





- Integer比较时：
  - Integer与int用==比较时，值相同则返回true，此时java会对integer拆包
  - 非new与new的Integer比较时，返回false
  - 两个非new的Integer比较时，值在-128~127之间时，返回true（涉及到装包（valueOf）的原理，-128~127会直接返回cache里面的对象，而不会new一个）



### 11、String，StringBuffer，StringBuilder的区别？

String是一个字符串的常量，而StringBuffer是字符串变量

- String是常量，因此编译器可以将一个String对象进行共享，当修改String对象，如下：

  ```java
  String str = "qia"
  str = str + "ningmeng"
  ```

  编译器实际重新new了一个字符串，然后将其地址赋给str；而之前在堆上的字符串则被JVM的GC机制清理掉了

- StringBuffer字符串处理时，不生成新的对象，因此当需要增加，删除串时，具有更高的效率

- 修改字符串时，StringBuilder是线程不安全的，StringBuffer内置方法很多带有synchronized关键字，保证了其线程安全



### 12、如何跳出多重嵌套循环？

在某一层循环前添加标记，并在break后加上标记。

```java
jumpToLoop1:
for (int loop1 = 0; loop1 < 100; loop1++) {
    for (int loop2 = 0; loop2 < 100; loop2++) {
        for (int loop3 = 0; loop3 < 100; loop3++) {
            break jumpToLoop1;
        }
    }
}
```



### 13、Java正则表达式

基础替代符：

- 字符匹配与范围控制：

  . 
  
  \s  
  
  \S  
  
  \w  
  
  \W  
  
  []

- 数量控制符：

  ==?== 表示1个或0个。换句话说，表示要不然没有，要不然只有1个
  
  ==\*== 表示0个或多个。
  
  ==\+== 表示1个或多个。
  
  =={n}==	表示正好n个
  
  =={n,m}== 表示n到m个，这是一个左闭右闭区间
  
  =={n,}==	表示至少n个



​		??,  *?,  +?,  {n}?,  {n, m}?,  {n,}? 





​		?+,  *+,  ++,  {n}+,  {n, m}+,  {n,}+





- 位置匹配符

  ^ $ \b \B \G



组件：

- Pattern

  ```java
  // 静态方法，返回一个Pattern对象
  public static Pattern compile(String regex);
  public static Pattern compile(String regex, int flags);
  public Matcher matcher(charSequence input);
  public static boolean matches(String regex, CharSequence input);
  public String[] split(CharSequence input, int limit);
  ```

  一般用compile返回一个Pattern对象，然后调用matcher方法返回Matcher对象

- Matcher:

  ```java
  1)  public boolean find() 
  从字符串开头向前匹配，直到匹配不符合停止，不管是部分匹配还是全部匹配，都返回true，如果匹配到字符串结束，依然没有匹配当前正则，那么返回false
  2)  public String group()
  这个方法必须和find方法配合使用，单独使用就会报错，返回上一个find方法匹配到的内容。 
  3) public boolean matches() 
  当前正则是否匹配字符串整体， 匹配返回true， 不匹配返回false
  4)  public String group(int group)
    这个方法和正则分组有关系，详细内容将在下一小节正则分组中详细描述。 
  5) public int groupCount()
  返回上一次匹配中，分成了几个组 
  6) public Matcher reset(CharSequence input)
  根据输入字符串，重新生成一个Matcher对象 
  7) public Matcher reset()
  重置指针，从头开始匹配。 
  8) public Matcher region(int start, int end)
  指定头尾指针的位置，重置指针。只匹配指定范围内的字符串。 
  9) public int start()
  返回头指针的位置 
  10) public int start(int group)
  这个方法和上面那个重载方法，意义完全不同。 这个是返回 指定的分组，在上一次匹配操作后的头指针位置。
  11) public int end()
  返回上一次匹配的尾指针位置。 
  12) public int end(int group)
  返回 指定的分组，在上一次匹配操作后的尾指针位置 
  13) public String replaceFirst(String replacement)
  替换第一个匹配到的内容 
  14) public String replaceAll(String replacement)
  替换全部匹配的内容
  ```

- 一个分组示例：

  ```java
  // 目标字符串，目标是要提前IOPS，avg的值
  String s = " IOPS=2932, bw=329,and other number avg=399, savg=23029";
   
  // 正则。 
  String regular = "^.*IOPS=(\\d+)\\b.*\\bavg=(\\d+)\\b.*$";
  Matcher matcher = Pattern.compile(regular).matcher(s);
   
  if(matcher.find() && matcher.groupCount() == 2) {
      System.out.println(matcher.group(1));
      System.out.println(matcher.group(2));
  }
  ```

  - 对于正则表达式， ^ 匹配字符串开始位置； .* 匹配几个空格；IOPS=匹配IOPS=；\\\d+匹配2932，作为第1组内容；\\\b匹配2与，之间的位置；.* 匹配, bw=329,and other number ;\\\b匹配空格与a之间的位置; avg=匹配avg=; \\\d+匹配399；\\\b匹配9与逗号之间的位置； .*匹配， savg=23029; $匹配结束位置。

  - 调用一次find方法，将匹配到全部字符串内容； 整个匹配内容被分成了三组，第0组是全部字符串内容，第1组得到2932， 第2组内容是399； 调用groupCount，不包括第0组，所以结果为2



### 14、Java是如何支持正则表达式操作的？

Java中的String类提供了支持正则表达式操作的方法，包括：matches()、replaceAll()、replaceFirst()、split()。此外，Java中可以用Pattern类表示正则表达式对象，它提供了丰富的API进行各种正则表达式操作



### 15、枚举类

在某些情况下，一个类的对象时有限且固定的，如季节类，它只有春夏秋冬4个对象这种实例有限且固定的类，在 Java 中被称为枚举类；

在 Java 中使用 enum 关键字来定义枚举类，其地位与 class、interface 相同；

枚举类是一种特殊的类，它和普通的类一样，有自己的成员变量、成员方法、构造器 (只能使用 private 访问修饰符，所以无法从外部调用构造器，构造器只在构造枚举值时被调用)；

一个 Java 源文件中最多只能有一个 public 类型的枚举类，且该 Java 源文件的名字也必须和该枚举类的类名相同，这点和类是相同的；

使用 enum 定义的枚举类默认继承了 java.lang.Enum 类，并实现了 java.lang.Seriablizable 和 java.lang.Comparable 两个接口;

所有的枚举值都是 public static final 的，且非抽象的枚举类不能再派生子类；

枚举类的所有实例(枚举值)必须在枚举类的第一行显式地列出，否则这个枚举类将永远不能产生实例。列出这些实例(枚举值)时，系统会自动添加 public static final 修饰，无需程序员显式添加。



### 16、NIO

同步非阻塞的I/O模型

一个同步阻塞I/O编程模型：服务器，主线程阻塞的socket.accept()，出现新的连接就扔进线程池里。

​	存在问题（多连接数下存在问题，少连接下运行效率很好且编程简单）：

​	1、线程本身占用较大内存，像Java的线程栈，一般至少分配512K～1M的空间

​	2、线程切换成本很高（保存上下文等）



所有系统I/O分为两个阶段：等待就绪和操作（例如等待可读和真正读），而等待就绪这一阶段是**不使用CPU的**，而操作这一个阶段，数与memory copy，很快，基本不耗时

BIO中，“我要读”，NIO，“我可以读”，AIO，“读完了”

NIO一个重要的特点是：socket主要的读、写、注册和接收函数，在等待就绪阶段都是非阻塞的，真正的I/O操作是同步阻塞的（消耗CPU但性能非常高）

NIO的主要事件有几个：读就绪、写就绪、有新连接到来。











## 二、关键字

### 15、Syncronized关键字

- 内存可见性：
- 操作原子性：两个同步块只能串行进入

**锁的内存语义：**

- 释放锁时，JMM把线程对应的本地内存中的共享变量刷回主内存
- 获取锁时，JMM把线程对应本地内存设置为无效，从而临界区代码必须从主内存中读取共享变量

**JVM的处理流程：**

​	JVM通过进入和退出Monitor对象来实现方法同步和代码块的同步

- 代码块同步通过monitorenter和monitorexit实现，monitorenter编译后放在同步代码块的开始位置，monitorexist放在方法结束和异常处。任何对象都有一个monitor相关联
- 执行monitorenter，首先尝试获取对象的锁，如果没有锁定或已经拥有，则锁计数器+1，monitorexit则-1，计数器减到0就释放锁

​	对同一线程synchronized锁时可重入的

底层用mutex lock实现，而Java的线程映射到操作系统的原生线程上，阻塞或唤醒都会用户态与核心态切换；JDK1.6以后进行优化，阻塞之前先自旋等待一段时间

**Java对象头**

|   长度    |          内容          |          说明          |
| :-------: | :--------------------: | :--------------------: |
| 32/64 bit |       Mark Word        |  存储hashcode与锁信息  |
| 32/64 bit | Class Metadata Address | 存储对象类型数据的指针 |
| 32/64 bit |      Array Length      | 数组长度（如果是数组） |

**锁优化**

为了避免用户态与核心态的频繁切换，将锁分为四种状态：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态，锁可以升级但不能降级

**Mark Word里面锁标志位：**

| 轻量级 |  重量级   | GC标记 |     偏向锁     | 无锁 |
| :----: | :-------: | :----: | :------------: | :--: |
|   00   |    10     |   11   |       01       |  01  |
|  自旋  | 依赖mutex |        | 只比较ThreadID |  -   |

还有一个锁状态位，1bit，表明是否为偏向锁

1. **偏向锁：在只有一个线程执行同步块时提升性能**

   一个线程访问同步块并获取锁时，对象头和栈帧中的锁记录里会存储锁偏向的线程ID，线程在进入和退出只需简单地测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁，偏向锁只需在置换线程ID的时候使用一次CAS原子指令

   - 偏向锁获取：

     查看Mark Word状态-->测试线程ID-->CAS测试或直接进入代码块

   - 偏向锁释放：

     竞争出现才会释放锁，到达全局安全点时释放锁

2. **轻量级锁：线程近乎交替执行同步块时提升性能**

3. **重量级锁：多个线程同一时间访问同一个锁**

   由轻量级锁膨胀而成

   通过monitor及其底层的mutex lock（互斥锁实现），核心态与用户态切换，消耗时间

4. **锁的转化**

   ![preview](https://pic2.zhimg.com/v2-48d61b481f25319d76df689f8ab9bc6d_r.jpg)

   一个对象刚开始实例化，持有偏向锁

   一旦有第二个线程访问这一个对象，会看到对象上存在竞争（偏向锁不会自动释放），检查持有偏向锁的线程，如继续运行则升级为轻量级锁

   如果自旋超过一定次数，或者第三个线程来访，则升级为重量级



**其他锁优化**

锁消除、锁粗化、自旋锁与自适应自旋锁



**synchronized关键字使用方法**：

1. 修饰方法：对象锁

   ```java
   public synchronized void method(){}
   //等价于
   public void method(){
       synchronized(this){};
   }
   ```

2. 修饰静态方法：类锁

   ```java
   public synchronized static void method(){}
   //等价于
   public void method(){
       synchronized(obj.class){};
   }
   ```

3. 修饰run方法：与普通方法类似



### 16 、volatile关键字

volatile是Java提供的一种轻量级的同步机制

**JMM内存模型**：

​	线程有本地内存，保存了线程对主内存中共享变量的副本拷贝，线程不能直接读写主内存中的变量

**内存可见性：**

- 写volatile变量时，线程本地内存中的变量强制刷新到主内存中
- 写操作会导致其他线程中的缓存无效，当其需要读取时，必须到主存读取

**防止指令重排：**

- 若用volatile修饰共享变量，在编译时，会在指令序列中插入内存屏障来禁止特定类型的处理器重排序

- 重排序规则：

  　　1.当第二个操作是voaltile写时，无论第一个操作是什么，都不能进行重排序

    　　2.当地一个操作是volatile读时，不管第二个操作是什么，都不能进行重排序

    　　3.当第一个操作是volatile写时，第二个操作是volatile读时，不能进行重排序



### 17、synchronized 与 Lock区别

JMM中线程的六种状态：

![img](http://ata2-img.cn-hangzhou.img-pub.aliyun-inc.com/a3dcc274847c84bc44b9c3aa6d7f488e.jpg)

区分阻塞的三种状态：Waiting , Blocked, Timed_waiting

1.synchronized是java内置**关键字**，在jvm层面，Lock是个**java类**；

2.synchronized会**自动释放锁**(a 线程执行完同步代码会释放锁 ；b 线程执行过程中发生异常会释放锁)，Lock需在finally中**手工释放锁**（unlock()方法释放锁），否则容易造成线程死锁；

3.synchronized的锁**可重入、不可中断、非公平**，而Lock锁**可重入、可判断、可公平**（两者皆可）

Synchronized (简单情况)
- 优点：实现简单，语义清晰，便于JVM堆栈跟踪；加锁解锁过程由JVM自动控制，提供了多种优化方案。 
- 缺点：不能进行高级功能（定时，轮询和可中断等）。

Lock （复杂情况）
- 优点：可定时的、可轮询的与可中断的锁获取操作，提供了读写锁、公平锁和非公平锁　　 
- 缺点：需手动释放锁unlock，不适合JVM进行堆栈跟踪。



### 18、final关键字

[参考](https://zhuanlan.zhihu.com/p/33083924)

**final修饰变量：**

- 修饰基本类型变量，值不能改变
- 修饰引用类型变量，变量不能再次被赋值，变量引用的对象值可以改变

**final表示常量：**

```java
public static final int CONST = 5;
```

**final保证实例变量被初始化：**

防止出现NPE，final使用之前必须初始化，否则不能通过编译

**final修饰方法与类：**

- 修饰方法：子类无法修改方法，注意private的区别
- 修饰类：此类是密封，无法被继承，例如String类

变量不可变可以减少编程中出错的风险，特别是多线程环境下





## 三、面向对象

### 19、wait底层原理

[参考](https://blog.csdn.net/a460708485/article/details/82023662)

lock.wait()调用native方法wait(0)实现，前提是必须要持有该lock的monitor对象

_WaitSet  ：处于wait状态的线程，会被加入到wait set；
_EntryList：处于等待锁block状态的线程，会被加入到entry set；

ObjectMonitor对象的void wait(...)实现：

- 将当前线程封装成ObjectWaiter对象node
- 通过ObjectMonitor::AddWaiter方法将node添加到_WaitSet列表中
- 通过ObjectMonitor::exit方法释放当前的ObjectMonitor对象，这样其它竞争线程就可以获取该ObjectMonitor对象。

ObjectMonitor对象的void notify()实现：

- 如果当前_WaitSet为空，即没有正在等待的线程，则直接返回
- 通过ObjectMonitor::DequeueWaiter方法，获取_WaitSet列表中的第一个ObjectWaiter节点
- 根据不同的策略，将取出来的ObjectWaiter节点，加入到_EntryList，或者进行自旋

### 20、Java有哪些特性？举例多态？

三大特性：**封装、继承、多态**

Map<String,String> map1 = new HashMap<String,String>();
Map<String,String> map2 = new TreeMap<String,String>();

Map接口的不同实现方式

### 21、String为什么不可变

[参考](https://www.cnblogs.com/leskang/p/6110631.html)

String内部封装了一个final类型（不能改变引用）的char数组，且是private，没有提供set方法

```java
private final char[] value;
```

通过反射可以获取value的域，并修改默认的访问权限，进行修改



### 22、Object类的方法

```java
registerNatives()   //私有方法
getClass()    //返回此 Object 的运行类。
hashCode()    //用于获取对象的哈希值。
equals(Object obj)     //用于确认两个对象是否“相同”。
clone()    //创建并返回此对象的一个副本。 
toString()   //返回该对象的字符串表示。   
notify()    //唤醒在此对象监视器上等待的单个线程。   
notifyAll()     //唤醒在此对象监视器上等待的所有线程。   
wait(long timeout)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量前，导致当前线程等待。   
wait(long timeout, int nanos)    //在其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量前，导致当前线程等待。
wait()    //用于让当前线程失去操作权限，当前线程进入等待序列
finalize()    //当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。
```

十二种方法



### 23、重写与重载的区别

重载：

一个类中，多个同名的方法有不同的参数列表（**参数类型不同、参数个数不同甚至是参数顺序不同**），不能通过**返回类型来区别是否重载**，编译时多态

重写：

子类重写父类的方法，**方法名，参数列表，返回类型(除过子类中方法的返回值是父类中方法返回值的子类时)都相同 ，子类函数的访问修饰权限不能少于父类的**，运行时多态



### 24、static关键字？

**方便在没有创建对象的情况下来进行调用（方法/变量）**，只要类加载了就可以调用

（1）修饰方法：

​	静态方法来说，**==没有this==**，不能访问**类的非静态成员变量和非静态成员方法**，因为非静态成员方法/变量都是必须依赖具体的对象才能够被调用

​	main方法设置为静态因为程序在执行main方法的时候没有创建任何对象

（2）修饰变量：

​	静态变量被**==所有的对象所共享==**，在内存中只有一个副本，它当且仅当在类初次加载时会被初始化

（3）修饰代码块：

​	类初次被加载的时候，会按照static块的顺序来执行每个static块，并且只会执行一次



注意：static不能出现在方法内部即不能修饰局部变量



**Java中是否可以覆盖(override)一个private或者是static的方法？**

（1）子类继承之后，**private修饰的方法对子类也是不可见的**。那么如果子类声明了一个跟父类中定义为private一样的方法，那么编译器只当作是你子类自己新增的方法，并不能算是继承过来的

（2）内存上来说是不可以的



### 25、String类能被继承吗？

**不能，类含有final修饰符**，见问题18



### 26、静态变量存在哪里？

方法区，见深入理解JVM第二章



### 27、泛型

[参考](<https://www.cnblogs.com/coprince/p/8603492.html>)

1、**定义**：用来规定一个类、接口或方法所能接受的数据的类型

2、**好处**：

- 提高安全性：将运行期的错误转换到编译期
- 避免强制类型转换：List不使用泛型时，取出的对象都是Object类型

3、特性：

​	泛型只在编译阶段有效，在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦除

3、泛型类：

​	泛型的类型参数只能是类类型，不能是简单类型

​	不能对确切的泛型类型使用instanceof操作，例如 if(ex_num instanceof Generic<Number>){ }

4、泛型接口：

​	与泛型类相似

3、通配符：

- 无边界通配符：<?>
- 固定上边界通配符(<? extends E>)：接受指定类及其子类类型的数据
- 固定下边界通配符(<? super E>)：接受指定类及其父类类型的数据



## 五、多线程

### 1、创建线程的方式

- 继承Thread类

- 实现Runnable接口

### 2、start()和run()方法的区别

start()方法调用后程序加入就绪队列，然后通过类中的run方法执行该线程的内容

run()方法调用后，程序还是要在主线程内顺序执行，还是单线程的

### 3、Runnable接口和Callable接口

Runnable接口返回值是void类型，Callable中的call方法是有返回值的，是一个泛型，用于和Future，FutureTask配合使用获得异步执行结果

Callable+Future/FutureTask却可以获取多线程运行的结果，可以在等待时间太长没获取到需要的数据的情况下取消该线程的任务

### 4、CyclicBarrier和CountDownLatch的区别

（1）CyclicBarrier的某个线程运行到某个点上之后，该线程即停止运行，直到所有的线程都到达了这个点，所有线程才重新运行；CountDownLatch则不是，某线程运行到某个点上之后，只是给某个数值-1而已，该线程继续运行

（2）CyclicBarrier只能唤起一个任务，CountDownLatch可以唤起多个任务

（3）CyclicBarrier可重用，CountDownLatch不可重用，计数值为0该CountDownLatch就不可再用了

### 5、ThreadLocal

用ThreadLocal维护变量时，为每个使用该变量的线程提供独立的变量副本

内部实现：

- 线程内部维护ThreadLocalMap对象，是一个键值对，里面包含了若干个Entry(ThreadLocal, value)，Entry对Key的引用是弱引用，对Value引用是强引用，因此最好调用remove方法去掉对value的引用，但是java也会对这种情况进行优化

### 6、线程池

`java.util.concurrent.ThreadPoolExecutor`

调用ThreadPoolExecutor.submit(Runnable task)提交任务

三个核心的属性：

- 当前线程池大小

- 最大线程池代销（maximunPoolSize):线程池中允许存在的工作者线程的数量上限
- 核心线程大小（corePoolSize）：不大于最大线程池大小的工作者线程数量上限

运行数量少于corePoolSize，首选添加新线程，而不进行排队

运行数量等于或多于corePoolSize，首选将请求加入队列，而不是添加新线程

无法添加队列，则队列已满，此时创建新线程，除非创建线程超过了maximumPoolSize

**为什么要使用线程池**

 1、减少创建和销毁线程的次数，每个工作线程被重复利用

 2、根据系统的承受能力，调整线程池中工作线程的数目



## 六、集合

### 1、HashMap

摘自：[Java8 HashMap](https://zhuanlan.zhihu.com/p/21673805)

`java.util.Map`四个实现类：`HashMap` `Hashtable` `LinkedHashMap` `TreeMap`

**特点：**

1、HashMap: 根据键的hashcode存储；遍历顺序不确定；允许一条键的值为null；非线程安全

2、HashTable：映射功能与HashMap类似，是线程安全的，对方法加锁

3、LinkedHashMap：HashMap的子类，保存了插入的次序，遍历是有序的

4、TreeMap：能够将记录按照键排序，key值必须实现comparable接口或者在构造Treemap的时候传入了自定义的comparator

**实现结构：**

数组+链表+红黑树

![1564926537692](C:\Users\bingmeishi\AppData\Roaming\Typora\typora-user-images\1564926537692.png)

**数据底层存储**

（1）哈希类中有一个字段为Node[] table，即哈希桶数组，Node是HashMap的内部类，实现了Map.Entry接口

```java
static class Node implements Map.Entry<K, V>{
    final int hash;    //用来定位数组索引位置
    final K key;
    V value;
    Node<K,V> next;   //链表的下一个node
    Node(int hash, K key, V value, Node<K,V> next) { ... }
    ...
}
```

（2）HashMap采用链地址法，数组加链表的组合，当数据被Hash后，得到数组下标，把数据放在对应下标元素的链表上，（hash碰撞：有时两个key会定位到相同的位置）需要好的Hash算法和扩容机制。



**HashMap字段**

table的length，Load factor为负载因子(默认值是0.75)，threshold是HashMap所能容纳的最大数据量的Node，size为实际存储的键值对

table的length取2的n次方，主要为了扩容和取模运算时的优化，但是这样会增加哈希碰撞（不是素数），HashMap在定位hash索引时，加入了高位运算的过程

不管怎么设计，链表过长的风险依然存在，因此链表过长时，引入了红黑树



**功能实现**

**hash**

```java
取hashcode 高位运算 取模
static final int hash(Object key){
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);  //第一二步，hashcode，高位运算
}
static int indexFor(int h, int length) {
     return h & (length-1);  //第三步 取模运算
}
```

通过h & (table.length -1)来得到该对象的保存位，而HashMap底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当length总是2的n次方时，h& (length-1)运算等价于对length取模，也就是h%length，但是&比%具有更高的效率



**put方法**

1、判断键值对数组table[i]是否为空或为null，否则执行resize()进行扩容；

2、根据键值key计算hash值得到插入的数组索引i，如果table[i]==null，直接新建节点添加，转向6，如果table[i]不为空，转向3；

3、判断table[i]的首个元素是否和key一样，如果相同直接覆盖value，否则转向4，这里的相同指的是hashCode以及equals；

4、判断table[i] 是否为treeNode，即table[i] 是否是红黑树，如果是红黑树，则直接在树中插入键值对，否则转向5

5、遍历table[i]，判断链表长度是否大于8，大于8的话把链表转换为红黑树，在红黑树中执行插入操作，否则进行链表的插入操作；遍历过程中若发现key已经存在直接覆盖value即可；

6、插入成功后，判断实际存在的键值对数量size是否超多了最大容量threshold，如果超过，进行扩容。



**扩容机制**

本质上是用一个容量更大的数组来替代之前的小数组

在jdk1.7中会重新计算每一个Key的新位置

jdk1.8的优化：扩容为2的幂次方（扩大一倍），元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置，扩容时，只需要看看原来的hash值新增的那个bit是1还是0就好了；这样子省去了rehash的计算；另外，新增的1bit是0还是1可以认为是随机的，因此可以分散节点，同时，节点不会倒置

![preview](https://pic2.zhimg.com/a285d9b2da279a18b052fe5eed69afe9_r.jpg)





### 2、ConcurrentMap

`java.util.concurrent.ConcurrentMap`:HashMap的线程安全版本，读操作不加锁，写操作通过只对特定的key值加锁

通过锁分段技术保证高并发下的读操作，只对hash值在同一段中的数据才会有竞争

get操作没有加锁，key对应的value值是volatile修饰的，当出现有key值没有value时（失效），会加lock，等待value写入后再读取

并发度ConcurrentLevel设置为2的n次方，根据hash值前缀或者后缀快速定位所属的segment和线性链



### 3、HashMap的key为自定义类

重写hashcode()与equals()方法，因为Object默认的hashcode方法是内存地址，因此有可能两个包含完全相同字段的对象其内存地址是不同的

### **4、ArrayList与LinkedList**

ArrayList内存中是连续的，随机访问(get,set)更快，LinkedList基于链表，插入删除更快，前提是要移动到对应的index，否则数据量大时不如Arraylist的速度

## 七、JVM

### 1、GC

jvm 中，程序计数器、虚拟机栈、本地方法栈都是随线程而生随线程而灭，栈帧随着方法的进入和退出做入栈和出栈操作，实现了自动的内存清理，因此，内存垃圾回收主要集中于 java 堆和方法区中



#### 存活

**对象存活判断**

引用计数：对象有一个引用计数的属性，引用计数为0时可以回收，但是无法解决循环引用的问题

可达性分析：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则是不可用的，称为不可达对象

GC ROOTS包括：

- 虚拟机栈中引用的对象。
- 方法区中类静态属性实体引用的对象。
- 方法区中常量引用的对象。
- 本地方法栈中JNI引用的对象。



**引用种类**

强引用 弱引用 软引用 虚引用

**强引用**

​		类似Object obj = new Object()此类的引用

**软引用**

​		描述还有用但是非必须的对象，在将要发生内存溢出时，会将软引用对象进行二次回收，如果仍然没有足够内存再抛出异常，SoftReference类来实现

**弱引用**

​		强度比软引用更弱，只能生存到下一次垃圾收集发生之前，WeakReference

**虚引用**

​		最弱的引用关系，唯一目的是在对象被回收时收到一个系统通知，它仅仅是提供了一种确保对象被 finalize 以后，做某些事情的机制，PhantomReference



**判断对象是否需要回收**

​		对象死亡要经历两次标记过程：没有引用链时进行第一次标记，并进行筛选（有无finalize方法或该方法是否被执行过）

​		如果有必要执行，这个对象会被放到一个叫F-Queue的队列中去，并且由一个低优先级的Finallizer线程执行finalize方法，虚拟机不承诺等待该方法执行结束。

​		如果对象能够在finalize方法中重新与引用链上的对象建立关联，比如把this指针赋给某个类变量或者是一个对象的成员，就回被移出即将回收的集合



**回收方法区**

​		主要有两类：废弃常量和无用的类，废弃常量只要该常量没有和任何一个对象关联即可，而废弃类则比较严苛：

​		1、该类所有实例都已经被回收了

​		2、加载该类的ClassLoader被回收了

​		3、对应的Class对象没有在任一地方被引用



**垃圾回收算法**

**标记-回收算法：**

首先标记所有需要回收的对象，标记完成后同一回收所有被标记的对象

存在两个问题：1、效率问题，标记和清除的效率都不高	2、空间问题，标记清除后会产生大量的不连续空间碎片，空间碎片太多会导致需要分配较大对象时触发一次新的GC

**复制算法：**

将内存分为大小相等的两块，每次只使用其中一块，当一块用完了，将还存活的对象统一复制到另一块上，这样不用考虑内存碎片，分配对象也更加容易，只需移动堆顶指针就可以了

实际使用中通常将内存分为较大的Eden空间和两块较小的Survivor空间，回收时将Eden和Survivor中存活的对象一次性复制到另一个Survier中，但是需要其他内存来做担保

**标记-整理算法：**

标记过程相同，但是后序步骤不是直接清理，而是让所有存活的对象向某一侧移动，然后直接清理掉端外的内存

**分代收集算法：**

将内存划分为多块，新生代和老年代，不同年代采取不同的垃圾收集策略



#### 算法实现

**枚举根节点**

可达性分析必须在一个一致性快照中进行。执行系统停顿后，并不需要一个不漏检查所有执行上下文和全局引用，HotSpot中有OopMap数据结构来存放对象引用，GC在扫描过程中可以直接得到信息

**安全点**

只是在特定的时间点记录OopMap，安全点选择是以“是否具有让程序长时间执行的特征”（指令序列复用，例如方法跳转，循环跳转，异常跳转），只有具有这些功能指令才会触发GC

抢先式中断（中断所有线程，如果不在安全点上就继续跑）与主动式中断（线程轮询一个标志）

**安全区域**

安全区域是指一段代码片段中，引用关系不会改变。线程执行到安全区域时，首先标志自己已经进入，离开时检测GCRoots有没有完成根节点枚举



#### **垃圾收集器**

**Serial收集器**

单线程，在收集的过程中会暂停其他工作线程，新生代复制算法，老年代标记-整理算法

优点：简单高效。仍然是Client模式下的默认新生代收集器，没有线程交互，收集效率很高

**ParNew收集器**

Serial收集器的多线程版

单CPU环境下比Serial收集器差，两个CPU环境都不一定超过Serial（线程交互开销）

**Parallel Scavenge收集器**

新生代、复制算法、并行

目的是达到一个可控制的吞吐量，吞吐量指CPU用于运行用户代码的时间与CPU总消耗时间的比值

**Serial Old收集器**

Serial收集器的老年代版本，标记-整理算法，可作为CMS收集器的备案

**Parallel Old收集器**

Parallel Scavenge的老年代版本，多线程与标记-整理算法，在注重吞吐量以及CPU资源敏感场合可以用Parallel Scavenge算法加上Parallel Old算法

**CMS收集器**

以获得最短回收停顿时间为目标的收集器，重视服务的响应速度，基于标记-清除算法

包括四个步骤：1、初始标记 2、并发标记 3、重新标记 4、并发清除

初始标记，重新标记会Stop The World，初始标记只标记GC Roots能直接关联的对象，速度很快；并发标记就是标记引用链的过程；重新标记是修正用户程序继续运行中引用关系改变。整个过程中耗时最长的并发标记，并发清除都是与用户线程一同工作的

缺点：

1、对CPU资源非常敏感。占用一部分CPU资源而导致应用程序变慢，总吞吐量降低，默认启动的线程数（CPU+3）/4，虚拟机提供增量式的并发收集器，GC线程与用户线程交替运行，减少GC独占资源时间，但效果很一般

2、无法处理浮动垃圾，并发清理过程可能会有新垃圾产生。而且用户线程还要产生新的垃圾，所以需要还需要足够的空间提供给用户线程使用，如果失败，就会启动备案，利用Serial Old代替CMS

3、空间碎片

**G1收集器**

面向服务端应用的收集器，具备以下特点

1、并行与并发，利用多个CPU缩短STW的时间

2、分代收集，不需要依赖其他收集器

3、空间整合，整体上看是标记-整理算法，局部上看基于复制算法

4、可预测停顿：建立可预测的停顿时间模型

G1收集器将整个Java堆分为多个大小相等的区域（Region），新生代和老年代不是物理隔离，只是一部分region集合。

避免在Java堆进行全区域的垃圾收集，维护Region中垃圾堆积的价值大小，维护一个优先级列表，优先回收价值最大的堆

Region之间的对象引用以及其他收集器中新生代和老年代间的对象引用通过Remembered Set避免全堆扫描，在对引用类型的数据进行写入的时候会产生中断，检查引用的对象是否处在不同的Region中，如果是，就记录到Remembered Set中去

包括四个步骤：1、初始标记	2、并发标记	3、最终回收	4、筛选回收



#### 内存分配与回收

**对象优先分配在Eden区**

对象优先分配在Eden区，当Eden区内存不够时，发生一次MinorGC

**大对象直接进入老年代**

大对象是指需要很长连续空间的对象，最典型的是长字符串和数组，避免在Survivor和Eden区进行大量的内存复制

**长期存活的对象进入老年代**

虚拟机给每一个对象定义了一个年龄计算器，在Survivor区每熬过一次Minor GC，年龄增加1，增加到一定年龄，就可以晋升老年代

**动态对象年龄判定**

如果在Survivor空间中相同年所有对象大小总和超过Survivor一半，超过或等于该年龄的对象直接进入老年代

**老年代分配担保**

首先检查老年代最大连续空间是否大于新生代对象总大小，如果是Minor GC是安全的，如果不是，会检查是否允许担保失败，允许的话会检查老年代最大连续空间是否大于历代晋升老年代对象的平均大小



### 2、Java内存区域

运行时数据区域包括方法区、虚拟机栈、本地方法栈、堆、程序计数器



**线程私有的有**：程序计数器、虚拟机栈



#### 运行时数据区域

**程序计数器**

​		较少的内存区域，可以看作是当前线程所执行的字节码的行号指示器，概念模型中，字节码解释器就是通过改变计数器的值来选取下一条要执行的字节码指令

​		每个线程都有一个程序计数器，各线程计数器互不影响，独立存储

​		如果执行Java方法，计数器记录正在执行的虚拟机字节码指令地址；Native方法为空



**Java虚拟机栈**

​		虚拟机栈描述了Java 方法执行的内存模型，每个方法在调用时会创建一个栈帧，用于存储局部变量表，操作数栈等信息，方法从调用直至执行完成的过程，对应一个栈帧在虚拟机栈中从入栈到出栈的过程

​		局部变量表存储编译器可知数据类型（基础类型），对象引用类型，和returnAddress类型（返回地址），所需内存空间在编译期确定



**本地方法栈**

​		与虚拟机栈类似，为Native方法服务



**Java堆**

​		Java堆被所有线程所共享，唯一目的是存放对象实例。

​		堆是垃圾回收器管理的主要区域，从内存回收角度上看，堆可以细分为：新生代和老年代；再细致一点可以是：Eden空间，From Survivor空间，To Survivor空间；从内存分配的角度看，可能会划分一些线程私有的分配缓冲区（TLAB），Java堆物理上可以不连续



**方法区**

​		被所有线程共享，存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码

​		很多人把方法区称为永久代，这个区域垃圾回收的主要目标是常量池和类型的卸载



**运行时常量池**

​		Class文件中的常量池进入方法区的运行时常量池存放



**直接内存**

​		堆外分配内存，然后通过堆中的对象对其进行引用

​		例如NIO中，Native函数库直接分配堆外内存，再通过Java堆中的DirectByteBuffer进行引用，避免了数据的来回复制



#### 对象

**对象创建**

​		一个new指令，首先会检查指令的参数能否在常量池中定位到一个符号引用，并且这个符号引用代表的类是否已经被加载；通过类加载检测后，虚拟机会为新生对象分配内存；内存分配完成后，虚拟机将分配的内存空间都初始化为零值（不包括对象头）；接下来对对象做必要的设置（对象属于哪个类，如何找到元数据信息，对象哈希码。GC分代年龄等，这些信息存放在对象头中）

​		内存分配的方法有两种：1、指针碰撞（用过的内存与空闲的内存是分开的）	2、空闲列表（虚拟机维护一个可用内存的列表）。内存分配同时需要考虑线程安全问题，一种方法是同步处理（CAS失败重试的方法）另一种方法是分配的动作在不同空间上进行，TLAB，只有在TLAB耗尽后才进行同步



**对象内存布局**

​		**对象头 实例数据 对其填充**

​		**对象头**包含两部分信息，第一部分存储运行时数据（哈希码 GC分代 锁状态标志 线程持有的锁 偏向线程ID），Mark Word被设计为一个非固定的数据结构另一部分是类型指针，指向它的类元数据的指针。如果对象是数组，则必须在对象头中保存数组的长度

​		**实例数据**部分是对象真正存储的有效信息，代码中定义的各类字段

​		**对其填充**起着占位符的作用，有的内存管理系统要求对象的起始地址必须是8的整数倍（对象大小）



**对象访问定位**

​		（1）句柄访问，Java堆中会划分出一块内存作为句柄池，句柄中包含了实例数据与类型数据各自的具体信息，优点是对象地址改变不用改变reference地址

​		（2）直接访问：引用中存储的是对象地址，优点是速度更快

​		

​		



### 3 类加载过程



类从被加载到虚拟机内存中，到卸载出内存，生命周期包括：加载、验证、准备、解析、初始化、使用、卸载

**加载的时机**：

1、new getstatic putstatic invokestatic四条字节码指令时，如果类没有初始化，则先进行初始化

2、使用reflect包方法对类进行反射调用的时候

3、初始化一个类，当其父类还未初始化时

4、虚拟机启动时，执行主类会首先初始化

以上四种称为对类的主动引用，除此之外的引用不会触发初始化，被动引用

被动引用：1、子类引用父类的静态字段，不会导致父类初始化；2、通过数组定义来引用类，不会触发初始化；3、对常量的引用

接口的初始化与类相似，但是第三点不同吗，接口初始化时不要求父接口全部完成初始化



##### 3.1 加载

**加载完成以下三件事：**

（1）通过类的全限定名获得定义这个类的**二进制字节流**

（2）将字节流所代表的**静态存储结构**转化为方法区的**运行时数据结构**

（3）在内存中生成一个代表这个类的**java.lang.Class对象**，作为方法区数据的访问入口

对于（1），并没有指明来源于方式，因此，有各种方式：

- 从zip包，jar，ear，war
- 网络获取 applet
- 运行时计算生成，动态代理技术，“*$Proxy”
- 其他文件读取，典型是jsp文件

对于非数组类型，程序员可以自行编写类加载器loadclass()，控制字节流的获取方式

对于数组类型，创建过程：

（1）组件类型为引用类型：递归地采用加载过程，数组将会被相应的类加载器类名称空间上被标志

（2）组件类型为基本类型：数组会标记为与引导类加载器关联

##### 3.2 验证

目的：确保class文件的字节流中包含的信息符合当前虚拟机要求，并不会危害虚拟机安全

主要完成四个阶段的验证工作：

（1）文件格式规范：

​				魔数、主次版本号、常量中是否有不被支持的常量类型、

（2）元数据验证，字节码描述信息进行语义分析：

​				类是否有父类、是否继承了不允许被继承的类、如果不是抽象类，是否实现了要求实现的方法

（3）字节码验证：验证数据流，控制流，对类的方法体进行校验分析，确定程序语义是否合法、符合逻辑

​				操作数栈的数据类型能匹配指令代码序列，跳转不会跳到方法体以外的字节码指令上

（4）符号引用验证：对类自身以外的信息进行匹配验证

​				通过字符串描述的全限定名能否找到对应的类

​				指定类中是否存在符合方法的字段描述符

​				类，字段、方法的访问性

##### 3.3 准备

为类变量分配内存并设定初始值：

**类变量**：被static修饰的变量

初始值：除final以外，其他值有特定的初始值，如int为0，而final修饰的字段则会在类字段中表现为ConstantValue会被初始化为特定的值

##### 3.4 解析

**将常量池内的符号引用替换成直接引用的过程**

（1）符号引用：可以是任何形式的字面量，与虚拟机的内存布局无关，符号引用的字面量形式定义在java虚拟机规范的class文件格式中

（2）直接引用：直接指向目标的指针，相对偏移量或是间接定位到目标的句柄。与虚拟机实现的内存布局相关

对同一个符号引用进行多次解析请求很常见，此时虚拟机会缓存解析结果



解析方法主要针对 类或接口、字段、类方法、接口方法、方法类型、方法句柄、调用点限定符进行

**（1）类或接口解析：**

​		将D中的符号引用N解析为类或接口C的直接引用：

​				1、C不是数组类型，将C的全限定名传给D的类加载器加载

​				2、C是数组类型，且元素类型为引用对象，则会加载加载数组的元素类型，并由虚拟机生成一个代表数组维度与元素的数组对象

​				3、确认D是否有对C的访问权限，否则抛出**java.lang.IllegalAccessError**

**(2)   字段解析：**

​		首先会对字段所属类进行解析，解析成功则进行：

​				1、C本身包含简单名称与字段描述符都匹配的字段，则直接返回这个字段的直接引用

​				2、否则，如果C中实现了接口，则会按照继承关系从下往上递归搜索

​				3、否则，如果C不是OBJECT类，则会按照继承关系递归搜索父类

​				4、否则，抛出**java.lang.NoSuchFieldError**异常

​	**成功返回后查看权限**

**（3）类方法解析**：

​		首先会对方法所属类进行解析，解析成功则进行：

​				1、如果发现C是接口，则抛出**java.lang.InCompatibleClassChangeError**异常

​				2、在类C查找是否有匹配

​				3、在类C的父类中递归查找是否有匹配

​				4、否则，在类C实现的接口中递归查找，如果找到则说明C为抽象类，抛出**java.lang.AbstractMethodError**异常

​				5、否则抛出**java.lang.NoSuchMethodError**异常

​		**查找结束后验证权限**

**（4）接口方法解析：**

​		首先会对方法所属类进行解析，解析成功则进行：

​				1、如果发现C是类，则抛出**java.lang.InCompatibleClassChangeError**异常

​				2、在接口C中查找是否有匹配

​				3、在C的父接口中递归查找是否有匹配

​				4、否则查找结束，抛出**java.lang.NoSuchMethodError**异常

​		不存在访问权限的问题，接口中所有方法都是public



##### 3.5 初始化

​		初始化时执行类构造器<clinit>()的过程
​		<clinit>()方法是编译器自动收集类中所有类变量的赋值动作和静态语句块顺序所决定的

#### 3.6 类加载器

​		任意一个类，需要由类加载器和类本身确定其在虚拟机中的唯一性

​		从虚拟机角度讲，存在两种类加载器，一种是启动类加载器（Bootstrap ClassLoader）,c++实现，是虚拟机自身的一部分，另一种是其他加载器，Java实现，都继承自java.lang.ClassLoader

**双亲委派模型**

除了顶层的启动类加载器以外，其余加载器都应有自己的父类加载器，不是继承关系，而是组合关系

一个类加载器收到了类加载请求，会首先委派给父类加载器去完成，所有加载请求最终都会传送到启动类加载器，只有当父类无法完成请求时，子类才会尝试加载该类

好处：Java类和其类加载器有了一种优先级层次关系，例如Object类，位于rt.jar中，始终都会由启动类加载器加载



## 八、Redis

[见](https://zhuanlan.zhihu.com/p/57715124)

### 1、Redis数据类型

- String：这是最简单的类型，普通的 set 和 get，做简单的 KV 缓存，set，get
- Map：存储结构化的数据，比如一个对象（不能嵌套另外的对象），操作，hset，hget

```
hset person name bingo
hset person age 20
```



- List：List可以存一些表型数据，评论列表，粉丝列表

  List可以完成分页（lrange）

  也可以做一个简单的消息队列，从list头进入，从list尾出

```
lrange mylist 0 3
lpush mylist
rpop mylist
```



- set：无序集合，自动去重，可以做交集，并集，差集

```
sadd mySet 1
smembers mySet
sismember
srem mySet 1

sinter yourSet mySet
sunion
sdiff
```



- sorted set：是排序的 set，去重但可以排序

```
zadd board 85 zhangsan

zrevrange board 0 3
zrank board zhangsan
```



### 2、Redis特点

基于内存操作的Key-Value数据库，定期异步将数据刷回硬盘，性能出色

支持多种数据结构，单个value最大内存限制1GB

缺点：数据库容量容易受到内存的限制

### 3、缓存

缓存用途：高并发，高性能

高性能：对于复杂操作耗时查出来的结果，如果确定不再变化，存在缓存里可以大大提高响应时间

高并发：走内存，查询速度很快，qps可以达到mysql的几十倍

### 4、Redis和Memcached区别

**Redis支持复杂数据结构**

redis拥有更多的数据结构，可以构建更丰富的操作

**Redis原生支持集群模式**

Redis支持cluster模式，而Memcached没有原生集群模式，只能通过客户端实现分片写入数据

**性能对比**

Redis单核，memecached多核，存储小数据上来说redis效率更高（避免线程切换），而大数据上来说，memcached性能更高一点

### 5、Redis线程模型

单线程的文件事件处理器（file event handler），使用I/O多路复用机制监听多个socket，将产生事件的socket压入到内存队列，时间分配器根据socket事件类型选择对应的事件处理器

文件事件处理器：

1、多个socket 2、I/O多路复用 3、事件分派器 4、事件处理器（连接应答处理器 命令请求处理器 命令回复处理器）

redis服务器进程初始化时，将server socket的AE_READABLE事件与连接应答处理器关联

**建立连接**

客户端启动，向服务端请求建立连接，此时服务器server socket产生AE_READABLE事件，将socket压入内存队列，文件分派器拿到socket，交给连接应答处理器，连接应答处理器创建一个新的socket，并且将该socket的AE_READABLE事件与命令请求处理器关联

**数据修改**

客户端发送请求，刚创建的socket会产生AE_READABLE事件，交给命令请求处理器，命令请求处理器对内存操作进行K-V设置，完成后将socket的AE_WRITABLE事件与命令回复处理器关联

**返回数据**

客户端准备好接收结果了后，用于通信的socket会产生AE_WRITABLE事件，事件分派器会找到命令回复处理器，进行回复，同时解除AE_WRITABLE事件与命令回复处理器的关联



**单线程快的原因**

- 纯内存
- 非阻塞的I/O多路复用（我猜的是耗时的主要是网络I/O，CPU并不是性能瓶颈）
- C语言实现
- 单线程避免了频繁的上下文切换



### 6、Redis过期策略

过期策略：定期删除+惰性删除

定期删除：redis每隔100ms定期抽取一些设置了过期时间的key，不检查全部key的原因是对CPU负载太高

惰性删除：当获取某一个key的时候，redis会检查，如果过期了不会返回任何东西



这里有一个小问题，内存里的key可能会累积得越来越多，这时需要走内存淘汰机制

几种内存淘汰机制：

1、noevication：内存不足时，新写入操作报错

2、allkeys-lru：在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）。

3、allkeys-random：在**键空间**中，随机移除某个 key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊。

4、volatile-lru：在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）。

5、volatile-random：在**设置了过期时间的键空间**中，**随机移除**某个 key。

6、volatile-ttl：在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除。



### 7、Redis 高并发 和 高可用？

高并发主要依靠主从架构，一主多从，如果高并发的同时容纳大量数据，需要使用Redis集群；高可用，在主从的基础上加上哨兵，主备切换

#### 7.1 主从架构实现高并发

**redis replication核心机制**

- redis异步将新数据复制到slave节点，redis2.8后，slave节点会确认复制的数量
- 一个master连多个slave，slave也可以连其他的slave节点
- slave复制时，master节点正常工作
- slave复制时，不会暂停服务，用旧数据提供服务，但是复制完成后删除旧数据集的时候会暂停服务
- slave可以做横向扩容

**核心原理**

启动一个slave node  会发送一个PSYNC命令个master node，初次连接master node会触发一次全量复制，master会启动一个后台线程，生成一份RDB快照文件，同时将所有写命令缓存在内存中，RDB发送到SLave的时候，slave先落磁盘，然后将rdb读入到内存，同时master将缓存的写命令发送给slave

**断点传续**

网络断掉了，可以接着从上次复制的地方继续复制

master node会在内存里维护一个backlog，master 和 slave 都会保存一个 replica offset 还有一个 master run id，offset 就是保存在 backlog 中的。如果 master 和 slave 网络连接断掉了，slave 会让 master 从上次 replica offset 开始继续复制，如果没有找到对应的 offset，那么就会执行一次 `resynchronization`。

**无磁盘复制**

master直接在内存中创建rdb，发给slave，不会落磁盘

**过期key**

slave节点不会有过期key，只会等待master节点来过期，master节点过期后，会模拟一条删除指令给slave

**复制的完整流程**

slave启动，在本地保存master node的信息，包括host和ip

slavenode有个定时任务，每秒检测是否有新的masternode要连接和复制，如果有，就建立链接，ping master，如果有验证需要先口令验证，验证通过后就第一次全量复制，后续master则将写命令异步复制给slave



#### 7.2 哨兵集群实现高可用

哨兵在redis集群中主要有以下功能：

- 集群监控：负责监控master和slave是否正常工作
- 消息通知：如果某个节点或实例挂了，就报警
- 故障转移：master节点挂了，自动转移至slave节点
- 配置中心：故障转移发生了，会自动通知客户端新的master地址

哨兵本身也是分布式的，作为一个哨兵集群互相协同工作：

- 故障转移，判断一个节点是否宕机，需要大部分哨兵都同意才行
- 即使部分哨兵节点挂了，集群还是能正常工作，因为如果一个高可用保证的系统本身是单点的，就很坑爹

**核心知识**

至少需要3个实例，来保证自己的健壮性；不保证数据零丢失；

大多数majority概念，超过一半

如果两个哨兵，一个实例M1和哨兵S1所在机器挂掉了，即使另一个哨兵S2依然运行，但是因为majority=2，因此并不会执行故障转移，此时三个哨兵就可以了

**哨兵主备切换的数据丢失问题（两种情况）**

1、主备切换的数据丢失：master节点还没有复制完写命令就挂掉了

2、脑裂导致数据丢失：某个master突然脱离正常网络，但依然运行，哨兵认为master挂了，就选一个新的slave，这个时候集群里就会有两个master，client还未来得及切换的时候会向旧master写入一部分数据，但是旧master重新连接时会变成slave，这部分数据会被清空

只能通过一些参数减少数据的丢失问题，一段时间内master没有收到来自slave复制的ack就拒绝写请求

**sdown与odown**

sdown：主观宕机，一个哨兵觉得master宕机了，就主观宕机

odown：客观宕机，超过quorum数量的哨兵觉得master宕机了，就客观宕机

sdown，哨兵ping一个master超过最大等待时间，odown，收到了超过quorum数量的哨兵发来的宕机信息

**哨兵集群自动发现机制**

pub/sub系统实现，往`__sentinel__:hello`channel里发定期发送消息和消费消息

**Slave->Master选举**

一个master odown了，并且majority数量哨兵允许主备切换，那么某个哨兵就会来执行主备切换的操作，考虑：

1、跟master断开时长 2、slave优先级 3、复制offset 4、run id

验证断开时长，满足特定阈值后，进行下列筛选：

1、按slave优先级进行排序

2、slave优先级相同，看replica offset，offset越靠后，优先级越高

3、上述条件都相同，选run id 小的

**quorum和majority**

主备切换时，首先需要quorum哨兵认为odown，然后需要majority哨兵授权某一个哨兵去完成主备切换

configuratioon

完成主备切换的哨兵会更新自己的master监控配置，并更新一个version号



### 8、Redis持久化

两种持久化方式：

1、RDB文件快照，redis对数据周期性的持久化。

2、AOF：以append-only模式将写命令写入一个日志文件中。

可以用这些方法将数据写到磁盘里，并备份到其他地方（比如云端），如果出了问题，就可以重建数据

**RDB优缺点**

- RDB会生成多个数据文件，每一个数据文件都代表某一个时刻的Redis数据
- RDB对Redis对外提供的读写服务影响非常小，可以让redis保持高性能，因为redis主进程只需要fork一个子进程，让子进程进行磁盘I/O即可
- 基于RDB重启和恢复redis进程，更加快速
- 如果想尽可能丢失少的数据，那么rdb不如aof，rdb一般是每隔5分钟或者更长时间生成一次
- 如果数据文件特别大，可能会导致对客户端服务暂停

**AOF优缺点**

- 更好保护数据不丢失，一般aof每隔1s就会执行以
- 以append-only写入，无磁盘寻址的开销，写入性能很高
- aof日志文件即使过大，出现后台重写数据，也不会影响客户端读写
- aof开启，支持的qps比rdb低，因为执行更加频繁

**如何选择rdb与aof**

- 不要只用rdb，丢失数据
- 不要只用aof，因为rdb恢复更快，也更健壮（数据快照）
- 同时开启aof和rdb，用aof做数据恢复的第一 萱蕚，aof破坏或不可用时上rdb快速恢复



### 9、Redis集群模式

redis cluster：

- 自动将数据进行分片，每个master存放一些数据
- 提供内置高可用，部分master不可用，也可以继续工作

redis cluster模式，每隔redist开放两个端口，6379和加1W 16379，16379用来节点间通信，c用来故障检测，配置更新，故障转移。gossip协议，用来高效数据交换

#### 9.1 内部通信机制

**基本通信原理**

元数据维护一般有两种方式：集中式、Gossip协议，redis cluster用的是Gossip协议

集中式是将元数据信息存储在某个节点上。用一个服务（ZooKeeper）对所有元数据进行存储维护

优点：元数据读取更新时效性很好，一旦改变立即更新

gossip协议，所有节点都持有一份元数据，如果出现元数据变更，就不断将元数据发送给其他节点

优点：元数据更新比较分散，不集中在一个地方，更新会分散打在不同节点上，降低压力，但是这样子数据更新会滞后

**gossip协议**

包含多种消息 ping pong meet fail

- meet：某个节点发送meet给新节点，新节点就会和其他节点开始进行通信
- ping：每隔节点都会频繁给其他节点发送ping，其中包含自己的状态还有自己维护的集群元数据，互相ping
- pong：返回ping和meet，包含自己的状态信息和其他信息，也用于信息广播，更新
- fail：如果判断另一个节点fail，就发送fail给其他节点

ping需要携带元数据，如果ping得太频繁，可能加重负担

每隔节点每秒ping十次。选择5个最久没有ping的点，如果延时达到阈值会立即ping。每次ping会带上自己节点的信息和一些其他节点的信息

#### 9.2 分布式寻址算法

**一致性哈希**





**hash slot算法**

redis集群中每一个master都会存有部分slot，slot移动（这个成本很低）。而客户端的数据，对指定的数据 走同一个slot。一个节点宕机，不影响其他节点，key找hash slot 而不是机器



redis cluster高可用与主备切换

与哨兵类似，不管是判断宕机（主观宕机与客观宕机），选举（连接时长，offset）



### 10、Redis雪崩，穿透，击穿

**缓存雪崩**

缓存机器意外全盘宕机，数据库扛不住，宕机，DBA重启数据库，无限宕机

解决方案：

事前：redis高可用，主从+哨兵或者是redis cluster，避免全盘崩

事中：本地ehcache缓存+限流，避免MySQL挂掉

事后，redis持久化，重启从磁盘加载数据，快速恢复缓存

用户发送请求，系统先查本地ehcache，然后redis，如果ehcache和redis都没有，再走数据库。限流

**优点：数据库不会死，数据库不死，服务就不会断**



**缓存穿透**

一个系统，一秒5000个请求，其中4000个是黑客恶意攻击，缓存中查不到，数据库里也查不到

解决方法：设置一个空值到缓存，设置一个过期时间，相同的key到缓存时，都可以直接从缓存中拿数据



**缓存击穿**

某个key非常热点，访问频繁，失效瞬间，大量请求直接打到数据库

解决方法:1、热点永不过期 2、redis zookeeper的互斥锁，缓存构建完毕后，再释放锁



### 11、Redis并发竞争问题

多个系统实例更新某一个key

可以基于ZooKeeper实现分布式锁，每隔系统通过ZooKeeper获得分布式锁，只有一个实例在操作某一个key，别的不允许读和写

写之前，先判断一下value的时间戳是否比缓存的时间戳心，新的话就能够写



### 12、Redis实现分布式锁

**单机Redis实现分布式锁**

获取锁

```
SET resource_name my_random_value NX PX 30000
my_random_value 一个随机值，保证唯一性
NX 没有的时候才设置
PX 30000 过期时间30s
```

释放锁

delete相关的key值



**Redis集群实现分布式锁**

master挂掉的话了数据丢失，可能另一个客户端会持有相同的锁

RedLock算法：

1、获取unix时间戳，以毫秒为单位

2、依次从不同的master节点中，使用相同key和唯一性的value获取锁。当请求锁时，客户端应该设置一个网络连接和相应超时时间，超时时间应该小于锁的失效时间。

3、客户端使用当前时间减去开始获取锁时间（步骤1记录的时间）就得到获取锁使用的时间。**当且仅当从大多数**（N/2+1，这里是3个节点）**的Redis节点都取到锁，并且使用的时间小于锁失效时间时，锁才算获取成功**。

4、如果取到了锁，key的真正有效时间等于有效时间减去获取锁所使用的时间（步骤3计算的结果）。

5、获取锁失败，应该在所有Redis实例上进行解锁



## 九、MySql

### 1、innoDB与MyISAM区别

- innoDB支持事务处理，外键，行级锁
- MyISAM支持全文索引，查询效率上来说要高一些
- InnoDB不不保存表的具体行数，MyISAM会有一个额外的值保存行数
- InnoDB支持聚簇索引，数据文件与索引绑定在一起

如何选择：

- 支持事务的话，innodb
- 绝大部分读查询，myisam，读写比较多，
- 系统崩溃后，myisam恢复更加困难



### **2、事务**

事务是一组原子性的SQL查询，满足ACID特性:

1、原子性：一个事务被视为一个不可分割的最小工作单元，所有操作要么全部提交成功，要么全部失败回滚

2、一致性：数据库始终从一个一致性状态转换到另一个一致性状态

3、隔离性：一个事务所做的修改在最终提交之前，对其他事务是不可见的

4、持久性：一旦提交，所做的修改会永久保存至数据库中



**并发事务可能带来的问题**

脏读：一个事务对某一条数据进行修改，此时另一个事务读取了这条信息

不可重复读：一个事务在读取某些数据后的某个时间，再次读取以前读过的数据，却发现其读出的数据已经发生了改变、或某些记录已经被删除了

幻读 ：一个事务按相同的查询条件重新读取以前检索过的数据，其他事务插入了满足其查询条件的新数据



**隔离级别**

读未提交：事务中的修改即使没有提交，对其他事务也是可见的

读已提交：只能看见已经提交的事务

可重复读（MySQL默认隔离级别）：同一事务多个实例在并发读取数据时，看到相同的数据行

串行化：强制事务串行执行



### 3、MVCC

MVCC 是行级锁的一个变种，避免了加锁操作，因此开销更低，大多数都实现了非阻塞的读操作，写操作只锁定固定的行，通过保存数据在某个时间点的快照来实现

InnoDB的MVCC

通过在每行记录后面保存两个隐藏的列来实现，一个保存了行的开始时间，一个保存了行的过期时间（删除），时间指的是系统版本号，每开始一个新的事务，系统版本号会自动递增

在可重复读的隔离级别下：

SELECT: 只查找版本早于当前事务版本的数据行；行的删除版本要么未定义，要么大于当前事务版本号

INSERT: 新插入的每一行保存当前系统版本号作为行版本号

DELETE: 删除的每一行保存当前版本号作为删除标志

UPDATE: 插入一条新纪录，同时更新之前数据的删除版本号



### **4、索引**

见 高性能MySQL 阅读笔记



### **5、关系型数据库**

关系表的设计确保将信息分解成多个表，一类数据一个表。各表通过某些常用的值（关系）相关联

**外键：**某个表中的一列，它包含另一个表的主键值，定义了两个表的关系

**联结：**一种机制，用于在一条SELECT语句中关联表



### 6、读写分离，主从复制

[见](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/mysql-read-write-separation.md)

实现：一个主库挂多个从库，写只写主库，主库将数据同步到从库中

原理：主库写binlog，从库拷贝binlog，再串行执行binlog中的sql（重新执行一遍）；

两个问题：1、延时（从库串行执行sql，网络I/O） 2、主库宕机未同步

解决方法：1、半同步复制：主库写binlog立即同步，从库写入relay中继后发送一个ack，主库此时认为写操作文成。2、并行复制：从库开多线程，并行读取relaylog中不同库的日志



现实中，如果主从复制的延时无法接受，可以有以下解决方法：

1、分库，减少主库的写并发	2、改变代码	3、如果真的立马需要查询，就直接读主库



### 7、分库分表

[见](https://github.com/doocs/advanced-java/blob/master/docs/high-concurrency/database-shard.md)

**为什么？**

支持高并发，增加sql执行效率、磁盘容量

单表数据量太大，会影响SQL性能，分表是将一个表的数据放到多个表中，查询的时候差一个表就可以了

单库不能支持太高的并发，因此分库，访问的时候查询一个库就可以了

**分库分表中间件？**

ShardingSphere，client层方案，不用部署，运维成本低，不需要代理层的二次转发请求，性能很高

MyCat，proxy 层方案，好处在于对于各个项目是透明的，但是运维成本高

**如何水平拆分或垂直拆分？**

水平拆分：把一个表的数据给弄到多个库的多个表里去，但是每个库的表结构都一样，只不过每个库表放的数据是不同的，所有库表的数据加起来就是全部数据，用更多的库来抗高并发

垂直拆分：把一个有很多字段的表给拆分成多个表，或者是多个库上去。每个库表的结构都不一样，每个库表都包含部分字段。会将较少的访问频率很高的字段放到一个表里去，然后将较多的访问频率很低的字段放到另外一个表里去，能够缓存更多的数据

那些中间件可以做到你分库分表之后，中间件可以根据你指定的某个字段值，比如说 userid，自动路由到对应的库上去，然后再自动路由到对应的表里去

水平拆分方式：

1、按照range分，容易产生热点问题，扩容简单

2、按照hash分，平均每个库的压力，但是扩容起来复杂

**未分库的表如何分库分表？**

1、停机

2、双写迁移

**怎样处理分布式id主键？**

1、数据库自增id，往一个没有意义的表里插入一条没有意义的数据，然后获得主键，然后再写，或者用一些分布式Id生成的服务（ZooKeeper）但是高并发不行

2、设置一个sequence和一个其实id，假如八个节点，步长就是8，但是增加节点时可能不好搞

3、UUID，优点本地生成，不走数据库，但是UUID太长，作为主键性能很差，且无序。

4、获得系统当前时间，并发大时不行

5、snowflake算法：64位的long型，1bit不用（统一为0），41bit时间戳，10bit序列号，12bit同一毫秒内不同id数





## 十、ZooKeeper

参照 Zookeeper-分布式一致性



## 十一、Dubbo



## 十二、Spring相关

### 1、IOC

[见](https://zhuanlan.zhihu.com/p/25459839)

**IOC原理概述**

需要对象的时候需要自己new出来，某些时候坏编程习惯导致无法回收。并且也违反了松耦合，少入侵的原则。最开始有一种解决方法是创建一个接口对象，然后根据需要改变其实现类，但是这样子修改实现类的时候需要修改内部代码

可以通过反射，在运行的时候动态生成某些类，将使用哪个类放在配置文件中，通过配置文件来对接口与实现类进行解耦合

Spring的IOC也是一个对象，可以进行对其他对象的控制，包括初始化，创建，销毁等。需要将要创建的类以及其依赖的类通过配置文件告诉spring。通过IOC可以促进松耦合，这种方式使一个对象依赖的对象以被动的方式传入

```xml

<bean name="accountDao" class="com.zejian.spring.springIoc.dao.impl.AccountDaoImpl"/>

<bean name="accountService" class="com.zejian.spring.springIoc.service.impl.AccountServiceImpl">
    <!-- 注入accountDao对象,需要set方法-->
    <property name="accountDao" ref="accountDao"/>
</bean>
```

使用这些类需要利用Spring提供的核心类，ApplicationContext，通过该类去加载已声明好的配置文件，然后便可以获取到我们需要的类了。

ClassPathXmlApplicationContext去加载spring的配置文件，

**Spring容器装配Bean（XML配置与注解）**

通过xml对bean进行声明和管理，每一个bean标签都代表着需要被创建的对象并通过property标签可以为该类注入其他依赖对象

### 2、AOP

[见](https://zhuanlan.zhihu.com/p/25522841)

3、



## 十三、其他

### 1、copy-on-write

[见](http://wsfdl.com/algorithm/2016/09/29/Copy_on_write.html)

​	资源管理方面的一种优化技术，假定多方需要使用同一个资源时，没有必要为每一方都创建该资源的一个完整的副本，反而令多方共享这个资源，当某方需要修改资源的某处时，利用引用计数，把该处复制一个副本，再把跟新的内容写入该副本中，从而节省创建多个完整副本时带来的空间和时间上的开销。

​	fork技术也是这样的，fork 之后的父进程和子进程完全共享数据段、代码段、堆和栈等的完全副本，而且内核将共享的地址空间的访问权限改变为只读，如果父进程和子进程中的任何一个试图修改这些区域，则内核修改区域的那块内存制作一个副本