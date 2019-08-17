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

[转自](https://mp.weixin.qq.com/s?__biz=MzI4NDY5Mjc1Mg==&mid=2247483952&idx=1&sn=ea12792a9b7c67baddfaf425d8272d33&chksm=ebf6da4fdc815359869107a4acd15538b3596ba006b4005b216688b69372650dbd18c0184643&scene=21#wechat_redirect)

jvm 中，程序计数器、虚拟机栈、本地方法栈都是随线程而生随线程而灭，栈帧随着方法的进入和退出做入栈和出栈操作，实现了自动的内存清理，因此，内存垃圾回收主要集中于 java 堆和方法区中



**对象存活判断**

引用计数：对象有一个引用计数的属性，引用计数为0时可以回收，但是无法解决循环引用的问题

可达性分析：从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则是不可用的，称为不可达对象

GC ROOTS包括：

- 虚拟机栈中引用的对象。
- 方法区中类静态属性实体引用的对象。
- 方法区中常量引用的对象。
- 本地方法栈中JNI引用的对象。





**垃圾回收算法**







## 八、Redis

### 1、Redis数据类型

- String：这是最简单的类型，普通的 set 和 get，做简单的 KV 缓存，set，get

- Map：存储结构化的数据，比如一个对象（不能嵌套另外的对象），操作，hset，hget

- List：List可以存一些表型数据，评论列表，粉丝列表

  List可以完成分页（lrange）

  也可以做一个简单的消息队列，从list头进入，从list尾出

```
lrange mylist 0 3
lpush mylist
rpop mylist
```



- set：无序集合，自动去重，可以做交集，并集，差集
- sorted set：是排序的 set，去重但可以排序