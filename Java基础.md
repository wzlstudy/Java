# 线程

## 1.线程的创建

**创建多线程的方式一：继承于Thread类**

1. 创建一个继承于Thread类的子类
2. 重写Thread类的run() --> 将此线程执行的操作声明在run()中
3. 创建Thread类的子类的对象
4. 通过此对象调用start()

```java
public class ExtendThread {
    public static void main(String[] args) {
        //调用该类的start方法启动线程
        MyTh1 myTh = new MyTh1();
        myTh.start();
    }
}

class MyTh1 extends Thread{
    /**
     * 重写run方法
     * 在run方法内写逻辑代码
     */
    @Override
    public void run() {
        for (int i = 0; i < 10; i++) {
            if (i%2==0){
                System.out.println(i);
            }
        }
    }
}
```

**创建多线程的方式二：实现Runnable接口**

1. 创建一个实现了Runnable接口的类
2. 实现类去实现Runnable中的抽象方法：run()
3. 创建实现类的对象
4. 将此对象作为参数传递到Thread类的构造器中，创建Thread类的对象
5. 通过Thread类的对象调用start()

```java
public class ImpRunnable {
    public static void main(String[] args) {
        MyThread2 myThread = new MyThread2();
        Thread thread = new Thread(myThread);
        thread.start();
    }
}

class MyThread2 implements Runnable{
    @Override
    public void run() {

    }
}
```

比较创建线程的两种方式。
开发中：优先选择：实现Runnable接口的方式
原因：1. 实现的方式没有类的单继承性的局限性
           2.实现的方式更适合来处理多个线程有共享数据的情况。
联系：public class Thread implements Runnable
相同点：两种方式都需要重写run(),将线程要执行的逻辑声明在run()中。
**创建线程的方式三：实现Callable接口。 --- JDK 5.0新增**

```java
//1.创建一个实现Callable的实现类
class NumThread implements Callable{
    //2.重写call方法，将此线程需要执行的操作声明在call()中
    @Override
    public Object call() throws Exception {
        int sum = 0;
        for (int i = 1; i <= 100; i++) {
            if(i % 2 == 0){
                System.out.println(i);
                sum += i;
            }
        }
        return sum;
    }
}

public class ThreadNew {
    public static void main(String[] args) {
        //3.创建Callable接口实现类的对象
        NumThread numThread = new NumThread();
        //4.将此Callable接口实现类的对象作为参数传递到FutureTask构造器中，创建FutureTask的对象
        FutureTask futureTask = new FutureTask(numThread);
        //5.将FutureTask的对象作为参数传递到Thread类的构造器中，创建Thread对象，并调用start()
        new Thread(futureTask).start();
		//new Thread(futureTask).start(); 同一个futureTask对象被多个线程调用只会执行一次，会将结果放到缓存，下次再调用这个对象时会直接返回结果
        try {
            //6.获取Callable中call方法的返回值
            //get()方法的返回值即为FutureTask构造器参数Callable实现类重写的call()的返回值。
            Object sum = futureTask.get();
            System.out.println("总和为：" + sum);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```


如何理解实现Callable接口的方式创建多线程比实现Runnable接口创建多线程方式强大？
1. call()可以有返回值的。
2. call()可以抛出异常，被外面的操作捕获，获取异常的信息
3. Callable是支持泛型的

**创建线程的方式四：使用线程池(1.5新增)**

```java
//1. 提供指定线程数量的线程池
ExecutorService service = Executors.newFixedThreadPool(10);
ThreadPoolExecutor service1 = (ThreadPoolExecutor) service;
//设置线程池的属性
//        System.out.println(service.getClass());
//        service1.setCorePoolSize(15);
//        service1.setKeepAliveTime();

//2.执行指定的线程的操作。需要提供实现Runnable接口或Callable接口实现类的对象
service.execute(new NumberThread());//适合适用于Runnable

//        service.submit(Callable callable);//适合使用于Callable
//3.关闭连接池
service.shutdown();
```

```java
//源码
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

创建一个线程池，该线程池复用固定数量的线程去操作一个共享的无界队列； 在任何时刻，最多只有nThreads的线程是处于可处理任务的活跃状态。 当所有的线程都处于活跃状态（在处理任务），如果提交了额外的任务，它将会在队列中等待，直到有线程可用。 如果线程在执行期间由于失败而终止，如果需要的话，一个新的线程将会取代它执行后续任务。 线程池中的线程将会一直存在，直到显示的关闭。

线程的数量是固定的，但是队列大小是无界的（Integer.MAX_VALUE足够大，大到可以任务无界。）

注意这里用的队列：LinkedBlockingQueue，默认队列大小为Integer的最大值。
```

好处：
1.提高响应速度（减少了创建新线程的时间）
2.降低资源消耗（重复利用线程池中线程，不需要每次都创建）
3.便于线程管理
     corePoolSize：核心池的大小
     maximumPoolSize：最大线程数
     keepAliveTime：线程没有任务时最多保持多长时间后会终止

## 2.Thread中的常用方法

1. start():启动当前线程；调用当前线程的run()
2. run(): 通常需要重写Thread类中的此方法，将创建的线程要执行的操作声明在此方法中
3. currentThread():静态方法，返回执行当前代码的线程
4. getName():获取当前线程的名字
5. setName():设置当前线程的名字
6. yield():释放当前cpu的执行权
7. join():在线程a中调用线程b的join(),此时线程a就进入阻塞状态，直到线程b完全执行完以后，线程a才
          结束阻塞状态。
8. stop():已过时。当执行此方法时，强制结束当前线程。
9. sleep(long millitime):让当前线程“睡眠”指定的millitime毫秒。在指定的millitime毫秒时间内，当前
                         线程是阻塞状态。
10. isAlive():判断当前线程是否存活

## 3.线程的优先级

1.线程的优先级：
   MAX_PRIORITY：10
   MIN _PRIORITY：1
   NORM_PRIORITY：5  -->默认优先级
2.如何获取和设置当前线程的优先级：
  getPriority():获取线程的优先级
  setPriority(int p):设置线程的优先级

说明：高优先级的线程要抢占低优先级线程cpu的执行权。但是只是从概率上讲，高优先级的线程高概率的情况下被执行。并不意味着只有当高优先级的线程执行完以后，低优先级的线程才执行。

## 4.同步机制解决线程的安全问题

 方式一：同步代码块

```
 synchronized(同步监视器){
     //需要被同步的代码
 }
```

 说明：1.操作共享数据的代码，即为需要被同步的代码。  -->不能包含代码多了，也不能包含代码少了。
            2.共享数据：多个线程共同操作的变量。比如：ticket就是共享数据。
            3.同步监视器，俗称：锁。任何一个类的对象，都可以充当锁。
            要求：多个线程必须要共用同一把锁。

**使用同步代码块解决继承Thread类的方式的线程安全问题**

**说明：在继承Thread类创建多线程的方式中，慎用this充当同步监视器，考虑使用当前类充当同步监视器。**

**补充：在实现Runnable接口创建多线程的方式中，我们可以考虑使用this充当同步监视器。**

 方式二：同步方法。
 如果操作共享数据的代码完整的声明在一个方法中，我们不妨将此方法声明同步的。

**使用同步方法解决实现Runnable接口的线程安全问题：**
**关于同步方法的总结：**
          1. **同步方法仍然涉及到同步监视器，只是不需要我们显式的声明**
          2. **非静态的同步方法，同步监视器是：this**
                **静态的同步方法，同步监视器是：当前类本身**

方式三：Lock锁  --- JDK5.0新增

```java
调用锁定方法lock()
lock.lock();
调用解锁方法：unlock()
lock.unlock();
```

面试题：synchronized 与 Lock的异同？相同：二者都可以解决线程安全问题
不同：synchronized机制在执行完相应的同步代码以后，自动的释放同步监视器
           Lock需要手动的启动同步（lock()），同时结束同步也需要手动的实现（unlock()）

优先使用顺序：
Lock  <------  同步代码块（已经进入了方法体，分配了相应资源） <------ 同步方法（在方法体之外）

死锁：
1.死锁的理解：
不同的线程分别占用对方都在等待对方放弃自己需要的同步资源，
2.说明：
1）出现死锁后，不会出现异常，不会出现提示，只是所有的线程都处于阻塞状态，无法继续
2）我们使用同步时，要避免出现死锁。
优缺点：同步的方式，解决了线程的安全问题。---好处
       操作同步代码时，只能有一个线程参与，其他线程等待。相当于是一个单线程的过程，效率低。 ---局限性

## 5.线程通信

涉及到的三个方法：
wait():一旦执行此方法，当前线程就进入阻塞状态，并释放同步监视器。
notify():一旦执行此方法，就会唤醒被wait的一个线程。如果有多个线程被wait，就唤醒优先级高的那个。
notifyAll():一旦执行此方法，就会唤醒所有被wait的线程。

说明：
1.wait()，notify()，notifyAll()三个方法必须使用在同步代码块或同步方法中。
2.wait()，notify()，notifyAll()三个方法的调用者必须是同步代码块或同步方法中的同步监视器。
   否则，会出现IllegalMonitorStateException异常
3.wait()，notify()，notifyAll()三个方法是定义在java.lang.Object类中。

面试题：sleep() 和 wait()的异同？
1.相同点：一旦执行方法，都可以使得当前的线程进入阻塞状态。
2.不同点：1）两个方法声明的位置不同：Thread类中声明sleep() , Object类中声明wait()。
             2）调用的要求不同：sleep()可以在任何需要的场景下调用。 wait()必须使用在同步代码块或同步方法中。
             3）关于是否释放同步监视器：如果两个方法都使用在同步代码块或同步方法中，sleep()不会释放锁，wait()会释放锁。

# 字符串

## 1.基本概念

String:字符串，使用一对""引起来表示。
    1.String声明为final的，不可被继承
    2.String实现了Serializable接口：表示字符串是支持序列化的。
                  实现了Comparable接口：表示String可以比较大小
    3.String内部定义了final char[] value用于存储字符串数据
   4.String:代表不可变的字符序列。简称：不可变性。
       体现：1.当对字符串重新赋值时，需要重新指定内存区域赋值，不能使用原有的value进行赋值。
   5.当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
  6.当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value  进行赋值。
  7.通过字面量的方式（区别于new）给一个字符串赋值，此时的字符串值声明在字符串常量池中。
  8.字符串常量池中是不会存储相同内容的字符串的。

String的实例化方式：
    方式一：通过字面量定义的方式
    方式二：通过new + 构造器的方式
    面试题：String s = new String("abc");方式创建对象，在内存中创建了几个对象？
            两个:一个是堆空间中new结构，另一个是char[]对应的常量池中的数据："abc"

```java
public void test2(){
    //通过字面量定义的方式：此时的s1和s2的数据javaEE声明在方法区中的字符串常量池中。
    String s1 = "javaEE";
    String s2 = "javaEE";
    //通过new + 构造器的方式:此时的s3和s4保存的是地址值，是数据在堆空间中开辟空间以后对应的地址值。
    String s3 = new String("javaEE");
    String s4 = new String("javaEE");

    System.out.println(s1 == s2);//true
    System.out.println(s1 == s3);//false
    System.out.println(s1 == s4);//false
    System.out.println(s3 == s4);//false

    System.out.println("***********************");
    Person p1 = new Person("Tom",12);
    Person p2 = new Person("Tom",12);

    System.out.println(p1.name.equals(p2.name));//true
    System.out.println(p1.name == p2.name);//true

    p1.name = "Jerry";
    System.out.println(p2.name);//Tom
}
```

**结论**：
    **1.常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量。**
    **2.只要其中有一个是变量，结果就在堆中。**
    **3.如果拼接的结果调用intern()方法，返回值就在常量池中**

## 2.类型转换

String 与 byte[]之间的转换
    编码：String --> byte[]:调用String的getBytes()
    解码：byte[] --> String:调用String的构造器

    编码：字符串 -->字节  (看得懂 --->看不懂的二进制数据)
    解码：编码的逆过程，字节 --> 字符串 （看不懂的二进制数据 ---> 看得懂）
    说明：解码时，要求解码使用的字符集必须与编码时使用的字符集一致，否则会出现乱码。
String 与 char[]之间的转换

    String --> char[]:调用String的toCharArray()
    char[] --> String:调用String的构造器
String 与基本数据类型、包装类之间的转换。

    String --> 基本数据类型、包装类：调用包装类的静态方法：parseXxx(str)
    基本数据类型、包装类 --> String:调用String重载的valueOf(xxx)
## 3.常用方法

替换：
String replace(char oldChar, char newChar)：返回一个新的字符串，它是通过用 newChar 替换此字符串中出现的所有 oldChar 得到的。
String replace(CharSequence target, CharSequence replacement)：使用指定的字面值替换序列替换此字符串所有匹配字面值目标序列的子字符串。
String replaceAll(String regex, String replacement)：使用给定的 replacement 替换此字符串所有匹配给定的正则表达式的子字符串。
String replaceFirst(String regex, String replacement)：使用给定的 replacement 替换此字符串匹配给定的正则表达式的第一个子字符串。
匹配:
boolean matches(String regex)：告知此字符串是否匹配给定的正则表达式。
切片：
String[] split(String regex)：根据给定正则表达式的匹配拆分此字符串。
String[] split(String regex, int limit)：根据匹配给定的正则表达式来拆分此字符串，最多不超过limit个，如果超过了，剩下的全部都放到最后一个元素中。
boolean endsWith(String suffix)：测试此字符串是否以指定的后缀结束
boolean startsWith(String prefix)：测试此字符串是否以指定的前缀开始
boolean startsWith(String prefix, int toffset)：测试此字符串从指定索引开始的子字符串是否以指定前缀开始
boolean contains(CharSequence s)：当且仅当此字符串包含指定的 char 值序列时，返回 true
int indexOf(String str)：返回指定子字符串在此字符串中第一次出现处的索引
int indexOf(String str, int fromIndex)：返回指定子字符串在此字符串中第一次出现处的索引，从指定的索引开始
int lastIndexOf(String str)：返回指定子字符串在此字符串中最右边出现处的索引
int lastIndexOf(String str, int fromIndex)：返回指定子字符串在此字符串中最后一次出现处的索引，从指定的索引开始反向搜索
注：indexOf和lastIndexOf方法如果未找到都是返回-1
int length()：返回字符串的长度： return value.length
char charAt(int index)： 返回某索引处的字符return value[index]
boolean isEmpty()：判断是否是空字符串：return value.length == 0
String toLowerCase()：使用默认语言环境，将 String 中的所有字符转换为小写
String toUpperCase()：使用默认语言环境，将 String 中的所有字符转换为大写
String trim()：返回字符串的副本，忽略前导空白和尾部空白
boolean equals(Object obj)：比较字符串的内容是否相同
boolean equalsIgnoreCase(String anotherString)：与equals方法类似，忽略大小写
String concat(String str)：将指定字符串连接到此字符串的结尾。 等价于用“+”
int compareTo(String anotherString)：比较两个字符串的大小
String substring(int beginIndex)：返回一个新的字符串，它是此字符串的从beginIndex开始截取到最后的一个子字符串。
String substring(int beginIndex, int endIndex) ：返回一个新字符串，它是此字符串从beginIndex开始截取到endIndex(不包含)的一个子字符串。

## 4.String、StringBuffer、StringBuilder

String、StringBuffer、StringBuilder三者的异同？
    String:不可变的字符序列；底层使用char[]存储
    StringBuffer:可变的字符序列；线程安全的，效率低；底层使用char[]存储
    StringBuilder:可变的字符序列；jdk5.0新增的，线程不安全的，效率高；底层使用char[]存储

    源码分析：
    String str = new String();//char[] value = new char[0];
    String str1 = new String("abc");//char[] value = new char[]{'a','b','c'};
    
    StringBuffer sb1 = new StringBuffer();//char[] value = new char[16];底层创建了一个长度是16的数组。
    System.out.println(sb1.length());//
    sb1.append('a');//value[0] = 'a';
    sb1.append('b');//value[1] = 'b';
    
    StringBuffer sb2 = new StringBuffer("abc");//char[] value = new char["abc".length() + 16];
    
    //问题1. System.out.println(sb2.length());//3
    //问题2. 扩容问题:如果要添加的数据底层数组盛不下了，那就需要扩容底层的数组。
             默认情况下，扩容为原来容量的2倍 + 2，同时将原有数组中的元素复制到新的数组中。
    
            指导意义：开发中建议大家使用：StringBuffer(int capacity) 或 StringBuilder(int capacity)
StringBuffer的常用方法：
StringBuffer append(xxx)：提供了很多的append()方法，用于进行字符串拼接
StringBuffer delete(int start,int end)：删除指定位置的内容
StringBuffer replace(int start, int end, String str)：把[start,end)位置替换为str
StringBuffer insert(int offset, xxx)：在指定位置插入xxx
StringBuffer reverse() ：把当前字符序列逆转
public int indexOf(String str)
public String substring(int start,int end):返回一个从start开始到end索引结束的左闭右开区间的子字符串
public int length()
public char charAt(int n )
public void setCharAt(int n ,char ch)
总结：
    增：append(xxx)
    删：delete(int start,int end)
    改：setCharAt(int n ,char ch) / replace(int start, int end, String str)
    查：charAt(int n )
    插：insert(int offset, xxx)
    长度：length();
    遍历：for() + charAt() / toString()

对比String、StringBuffer、StringBuilder三者的效率：
从高到低排列：StringBuilder > StringBuffer > String

# 日期

## 1.基本类

java.util.Date类
           |---java.sql.Date类

1.两个构造器的使用

构造器一：Date()：创建一个对应当前时间的Date对象

Date date1 = new Date();

构造器二：创建指定毫秒数的Date对象

Date date2 = new Date(155030620410L);

2.两个方法的使用

toString():显示当前的年、月、日、时、分、秒

getTime():获取当前Date对象对应的毫秒数。（时间戳）

3.java.sql.Date对应着数据库中的日期类型的变量

如何实例化

java.sql.Date date3 = new java.sql.Date(35235325345L);

如何将java.util.Date对象转换为java.sql.Date对象

Date date6 = new Date();
java.sql.Date date7 = new java.sql.Date(date6.getTime());

## 2.jdk 8之前的日期时间的API测试

1. System类中currentTimeMillis();

2. java.util.Date和子类java.sql.Date

3. SimpleDateFormat

4. Calendar

   SimpleDateFormat的使用：SimpleDateFormat对日期Date类的格式化和解析
   1.两个操作：
   1.1 格式化：日期 --->字符串

```java
 //实例化SimpleDateFormat:使用默认的构造器
 SimpleDateFormat sdf = new SimpleDateFormat();
 //格式化：日期 --->字符串
 Date date = new Date();
 System.out.println(date);
 String format = sdf.format(date);
```

  1.2 解析：格式化的逆过程，字符串 ---> 日期

```java
//解析：格式化的逆过程，字符串 ---> 日期
String str = "19-12-18 上午11:43";
Date date1 = sdf.parse(str);
System.out.println(date1);
```

  2.SimpleDateFormat的实例化
  指定的方式格式化和解析：调用带参的构造器
  SimpleDateFormat sdf = new SimpleDateFormat("yyyyy-MM-dd hh-mm-ss");

## 3.jdk 8之后的日期类

LocalDate、LocalTime、LocalDateTime 的使用

说明：
    1.LocalDateTime相较于LocalDate、LocalTime，使用频率要高
    2.类似于Calendar

方法：

​	1.now():获取当前的日期、时间、日期+时间

​	2.of():设置指定的年、月、日、时、分、秒。没有偏移量

​	3.getXxx()：获取相关的属性

​	4.withXxx():设置相关的属性(体现不可变性)

​	5.DateTimeFormatter:格式化或解析日期、时间。类似于SimpleDateFormat

# 排序

一、说明：Java中的对象，正常情况下，只能进行比较：==  或  != 。不能使用 > 或 < 的
        但是在开发场景中，我们需要对多个对象进行排序，言外之意，就需要比较对象的大小。
        如何实现？使用两个接口中的任何一个：Comparable 或 Comparator

二、Comparable接口与Comparator的使用的对比：
   	Comparable接口的方式一旦一定，保证Comparable接口实现类的对象在任何位置都可以比较大小。
   	Comparator接口属于临时性的比较。

## 1.Comparable接口的使用：自然排序

​    1.像String、包装类等实现了Comparable接口，重写了compareTo(obj)方法，给出了比较两个对象大小的方式。
​    2.像String、包装类重写compareTo()方法以后，进行了从小到大的排列
   3. 重写compareTo(obj)的规则：
       如果当前对象this大于形参对象obj，则返回正整数，
       如果当前对象this小于形参对象obj，则返回负整数，
       如果当前对象this等于形参对象obj，则返回零。

   4. 对于自定义类来说，如果需要排序，我们可以让自定义类实现Comparable接口，重写compareTo(obj)方法。
       在compareTo(obj)方法中指明如何排序。

       ```java
       public int compareTo(Object o) {
           if(o instanceof Goods){
               Goods goods = (Goods)o;
               //方式一：
               if(this.price > goods.price){
                   return 1;
               }else if(this.price < goods.price){
                   return -1;
               }else{
                   //             return 0;
                   return -this.name.compareTo(goods.name);
               }
               //方式二：
               //             return Double.compare(this.price,goods.price);
           }
           //        return 0;
           throw new RuntimeException("传入的数据类型不一致！");
       }
       ```

## 2.Comparator接口的使用：定制排序

​    1.背景：
​    当元素的类型没有实现java.lang.Comparable接口而又不方便修改代码，
​    或者实现了java.lang.Comparable接口的排序规则不适合当前的操作，
​    那么可以考虑使用 Comparator 的对象来排序，java.util.Comparator是一个接口，可以实现该接口也可以创建该接口的匿名内部类
​    2.重写compare(Object o1,Object o2)方法，比较o1和o2的大小：
​    如果方法返回正整数，则表示o1大于o2；
​    如果返回0，表示相等；
​    返回负整数，表示o1小于o2。

```java
new Comparator(){
public int compare(Object o1, Object o2) {
		return 0;
     }
};
```

# 枚举

## 一、枚举类的使用

 * 1.枚举类的理解：类的对象只有有限个，确定的。我们称此类为枚举类
 * 2.当需要定义一组常量时，强烈建议使用枚举类
 * 3.如果枚举类中只有一个对象，则可以作为单例模式的实现方式。

## 二、如何定义枚举类

 * 方式一：jdk5.0之前，自定义枚举类
 * 方式二：jdk5.0，可以使用enum关键字定义枚举类
 * 使用enum关键字定义枚举类
    * 说明：定义的枚举类默认继承于java.lang.Enum类
    * 提供当前枚举类的对象时，多个对象之间用","隔开，末尾对象";"结束

## 三、Enum类中的常用方法

 * values()方法：返回枚举类型的对象数组。该方法可以很方便地遍历所有的枚举值。
 * valueOf(String str)：可以把一个字符串转为对应的枚举类对象。要求字符串必须是枚举类对象的“名字”。如不是，会有运行时异常：IllegalArgumentException。
 * toString()：返回当前枚举类对象常量的名称

## 四、使用enum关键字定义的枚举类实现接口的情况

 * 情况一：实现接口，在enum类中实现抽象方法
 * 情况二：让枚举类的对象分别实现接口中的抽象方法

```java
public enum ResultVo {
    ResultVo1(400, "success"),
    ResultVo2(500,"服务器错误");

    private int code;
    private String msg;

    public int getCode() {
        return code;
    }

    public void setCode(int code) {
        this.code = code;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }

    private ResultVo(int code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    @Override
    public String toString() {
        return "ResultVo{" +
                "code=" + code +
                ", msg='" + msg + '\'' +
                '}';
    }
}
```

# 集合

集合(collection+map)框架的概述
 * 1.集合、数组都是对多个数据进行存储操作的结构，简称Java容器。

 * 说明：此时的存储，主要指的是内存层面的存储，不涉及到持久化的存储（.txt,.jpg,.avi，数据库中）

 * 2.1 数组在存储多个数据方面的特点：

     > 一旦初始化以后，其长度就确定了。

     > 数组一旦定义好，其元素的类型也就确定了。我们也就只能操作指定类型的数据了。

     比如：String[] arr;int[] arr1;Object[] arr2;

 * 2.2 数组在存储多个数据方面的缺点：

     > 一旦初始化以后，其长度就不可修改。

     > 数组中提供的方法非常有限，对于添加、删除、插入数据等操作，非常不便，同时效率不高。

     > 获取数组中实际元素的个数的需求，数组没有现成的属性或方法可用

     > 数组存储数据的特点：有序、可重复。对于无序、不可重复的需求，不能满足。

     二、集合框架

     |----Collection接口：单列集合，用来存储一个一个的对象

     ​	|----List接口：存储有序的、可重复的数据。  -->“动态”数组

     ​		|----ArrayList、LinkedList、Vector

     ​	|----Set接口：存储无序的、不可重复的数据   -->高中讲的“集合”

     ​		|----HashSet、LinkedHashSet、TreeSet

     |----Map接口：双列集合，用来存储一对(key - value)一对的数据   -->高中函数：y = f(x)

     ​	|----HashMap、LinkedHashMap、TreeMap、Hashtable、Properties

     三、Collection接口中的方法的使用

     1.add(Object e):将元素e添加到集合coll中

     2.size():获取添加的元素的个数

     3.addAll(Collection coll1):将coll1集合中的元素添加到当前的集合中

     4.clear():清空集合元素

     5.isEmpty():判断当前集合是否为空

     6.contains(Object obj):判断当前集合中是否包含obj(判断时会调用obj对象所在类的equals())

     7.containsAll(Collection coll1):判断形参coll1中的所有元素是否都存在于当前集合中。

     8.remove(Object obj):从当前集合中移除obj元素。

     9.removeAll(Collection coll1):差集：从当前集合中移除coll1中所有的元素。

     10.retainAll(Collection coll1):交集：获取当前集合和coll1集合的交集，并返回给当前集合

     11.equals(Object obj):要想返回true，需要当前集合和形参集合的元素都相同。

     12.hashCode():返回当前对象的哈希值

     13.集合 --->数组：toArray()

     拓展：数组 --->集合:调用Arrays类的静态方法asList()

     14.iterator():返回Iterator接口的实例，用于遍历集合元素。

     使用iterator()需先创建该对象：Iterator iterator = coll.iterator();


## 1.List接口框架:

|----Collection接口：单列集合，用来存储一个一个的对象

|----List接口：存储有序的、可重复的数据。  -->“动态”数组,替换原有的数组

​       |----ArrayList：作为List接口的主要实现类；线程不安全的，效率高；底层使用Object[] elementData存储

​       |----LinkedList：对于频繁的插入、删除操作，使用此类效率比ArrayList高；底层使用双向链表存储

​       |----Vector：作为List接口的古老实现类；线程安全的，效率低；底层使用Object[] elementData存储

## 2.ArrayList的源码分析：

### 2.1 jdk7情况下：

ArrayList list = new ArrayList();//底层创建了长度是10的Object[]数组elementData

list.add(123);//elementData[0] = new Integer(123);

list.add(11);//如果此次的添加导致底层elementData数组容量不够，则扩容。

**默认情况下，扩容为原来的容量的1.5倍，即在原来的基础上再增加一半容量(ArrayList.length+ArrayList.length/2向下取整)，同时需要将原有数组中的数据复制到新的数组中。复制使用的是Arrays.copyOf()方法**

结论：建议开发中使用带参的构造器：ArrayList list = new ArrayList(int capacity)

### 2.2 jdk8中ArrayList的变化：

ArrayList list = new ArrayList();//底层Object[] elementData初始化为{}.并没有创建长度为10的数组

list.add(123);//第一次调用add()时，底层才创建了长度10的数组，并将数据123添加到elementData[0]

后续的添加和扩容操作与jdk 7 无异。

### 2.3 小结：

jdk7中的ArrayList的对象的创建类似于单例的饿汉式，而jdk8中的ArrayList的对象的创建类似于单例的懒汉式，延迟了数组的创建，节省内存。

## 3.LinkedList的源码分析：

LinkedList list = new LinkedList(); 内部声明了Node类型的first和last属性，默认值为null

list.add(123);//将123封装到Node中，创建了Node对象。

其中，Node定义为：体现了LinkedList的双向链表的说法

```java
private static class Node<E> {
     E item;
     Node<E> next;
     Node<E> prev;
     Node(Node<E> prev, E element, Node<E> next) {
     this.item = element;
     this.next = next;
     this.prev = prev;
    }
 }
```



## 4.Vector的源码分析：

jdk7和jdk8中通过Vector()构造器创建对象时，底层都创建了长度为10的数组。

在扩容方面，默认扩容为原来的数组长度的2倍。

面试题：ArrayList、LinkedList、Vector三者的异同？

同：三个类都是实现了List接口，存储数据的特点相同：存储有序的、可重复的数据

不同：见上

## 5.List接口中的常用方法

void add(int index, Object ele):在index位置插入ele元素
boolean addAll(int index, Collection eles):从index位置开始将eles中的所有元素添加进来
Object get(int index):获取指定index位置的元素
int indexOf(Object obj):返回obj在集合中首次出现的位置
int lastIndexOf(Object obj):返回obj在当前集合中末次出现的位置
Object remove(int index):移除指定index位置的元素，并返回此元素
Object set(int index, Object ele):设置指定index位置的元素为ele
List subList(int fromIndex, int toIndex):返回从fromIndex到toIndex位置的子集合

总结：常用方法
增：add(Object obj)
删：remove(int index) / remove(Object obj)
改：set(int index, Object ele)
查：get(int index)
插：add(int index, Object ele)
长度：size()
遍历：① Iterator迭代器方式(jdk 5.0 新增了foreach循环，用于遍历集合、数组)
                集合元素的遍历操作，使用迭代器Iterator接口
               1.内部的方法：hasNext() 和  next()
               2.集合对象每次调用iterator()方法都得到一个全新的迭代器对象，默认游标都在集合的第一个元素之前。
               3.内部定义了remove(),可以在遍历的时候，删除集合中的元素。此方法不同于集合直接调用remove()
           ② 增强for循环
           ③ 普通的循环

## 6.Set

Set接口的框架：
|----Collection接口：单列集合，用来存储一个一个的对象
      |----Set接口：存储无序的、不可重复的数据   -->高中讲的“集合”
          |----HashSet：作为Set接口的主要实现类；线程不安全的；可以存储null值
              |----LinkedHashSet：作为HashSet的子类；遍历其内部数据时，可以按照添加的顺序遍历
                                  对于频繁的遍历操作，LinkedHashSet效率高于HashSet.
          |----TreeSet：可以按照添加对象的指定属性，进行排序。

Set接口中没有额外定义新的方法，使用的都是Collection中声明过的方法。

要求：向Set(主要指：HashSet、LinkedHashSet)中添加的数据，其所在的类一定要重写hashCode()和equals()
要求：重写的hashCode()和equals()尽可能保持一致性：相等的对象必须具有相等的散列码
重写两个方法的小技巧：对象中用作 equals() 方法比较的 Field，都应该用来计算 hashCode 值。

一、Set：存储无序的、不可重复的数据
    以HashSet为例说明：

1. 无序性：不等于随机性。存储的数据在底层数组中并非按照数组索引的顺序添加，而是根据数据的哈希值决定的。
2. 不可重复性：保证添加的元素按照equals()判断时，不能返回true.即：相同的元素只能添加一个。

二、添加元素的过程：以HashSet为例：
我们向HashSet中添加元素a,首先调用元素a所在类的hashCode()方法，计算元素a的哈希值，此哈希值接着通过某种算法计算出在HashSet底层数组中的存放位置（即为：索引位置），判断数组此位置上是否已经有元素：
 |----如果此位置上没有其他元素，则元素a添加成功。 --->情况1
 |----如果此位置上有其他元素b(或以链表形式存在的多个元素），则比较元素a与元素b的hash值：
         |----如果hash值不相同，则元素a添加成功。--->情况2
         |----如果hash值相同，进而需要调用元素a所在类的equals()方法：
                |----equals()返回true,元素a添加失败
                |----equals()返回false,则元素a添加成功。--->情况3
 对于添加成功的情况2和情况3而言：元素a 与已经存在指定索引位置上数据以链表的方式存储。
 jdk 7 :元素a放到数组中，指向原来的元素。
 jdk 8 :原来的元素在数组中，指向元素a
 总结：七上八下
 HashSet底层：底层是HashMap，而HashMap底层是数组+链表的结构。

**LinkedHashSet的使用**
LinkedHashSet作为HashSet的子类，在添加数据的同时，每个数据还维护了两个引用，记录此数据前一个数据和后一个数据。
优点：对于频繁的遍历操作，LinkedHashSet效率高于HashSet

**TreeSet的使用**
1.向TreeSet中添加的数据，要求是相同类的对象。
2.两种排序方式：自然排序（实现Comparable接口） 和 定制排序（Comparator）
3.自然排序中，比较两个对象是否相同的标准为：compareTo()返回0.不再是equals().
4.定制排序中，比较两个对象是否相同的标准为：compare()返回0.不再是equals().

参考：https://blog.csdn.net/a1439775520/article/details/95373610

## 7.Map

一、Map的实现类的结构：
 |----Map:双列数据，存储key-value对的数据   ---类似于高中的函数：y = f(x)
        |----HashMap:作为Map的主要实现类；线程不安全的，效率高；存储null的key和value
             |----LinkedHashMap:保证在遍历map元素时，可以按照添加的顺序实现遍历。
                     原因：在原有的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素。
                     对于频繁的遍历操作，此类执行效率高于HashMap。
        |----TreeMap:保证按照添加的key-value对进行排序，实现排序遍历。此时考虑key的自然排序或定制排序
                     底层使用红黑树
        |----Hashtable:作为古老的实现类；线程安全的，效率低；不能存储null的key和value
             |----Properties:常用来处理配置文件。key和value都是String类型

HashMap的底层：数组+链表  （jdk7及之前）
                                数组+链表+红黑树 （jdk 8）

二、Map结构的理解：
①Map中的key:无序的、不可重复的，使用Set存储所有的key  ---> key所在的类要重写equals()和hashCode() （以HashMap为例）
②Map中的value:无序的、可重复的，使用Collection存储所有的value --->value所在的类要重写equals()
③一个键值对：key-value构成了一个Entry对象。
④Map中的entry:无序的、不可重复的，使用Set存储所有的entry

三、HashMap的底层实现原理？以jdk7为例说明：
    HashMap map = new HashMap():
    在实例化以后，底层创建了长度是16的一维数组Entry[] table。
    ...可能已经执行过多次put...
    map.put(key1,value1):
    首先，调用key1所在类的hashCode()计算key1哈希值，此哈希值经过某种算法计算以后，得到在Entry数组中的存放位置。
    如果此位置上的数据为空，此时的key1-value1添加成功。 ----情况1
    如果此位置上的数据不为空，(意味着此位置上存在一个或多个数据(以链表形式存在)),比较key1和已经存在的一个或多个数据的哈希值：
            ①如果key1的哈希值与已经存在的数据的哈希值都不相同，此时key1-value1添加成功。----情况2
            ②如果key1的哈希值和已经存在的某一个数据(key2-value2)的哈希值相同，继续比较：调用key1所在类的equals(key2)方法，比较：
                    ①如果equals()返回false:此时key1-value1添加成功。----情况3
                    ②如果equals()返回true:使用value1替换value2。
补充：①关于情况2和情况3：此时key1-value1和原来的数据以链表的方式存储。
②在不断的添加过程中，会涉及到扩容问题，当超出临界值(且要存放的位置非空)时，扩容。**默认的扩容方式：扩容为原来容量的2倍，并将原有的数据复制过来**。

jdk8 相较于jdk7在底层实现方面的不同：
1. new HashMap():底层没有创建一个长度为16的数组
2. jdk 8底层的数组是：Node[],而非Entry[]
3. 首次调用put()方法时，底层创建长度为16的数组
4. jdk7底层结构只有：数组+链表。jdk8中底层结构：数组+链表+红黑树。
   4.1 形成链表时，七上八下（jdk7:新的元素指向旧的元素。jdk8：旧的元素指向新的元素）
   4.2 当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8 且当前数组的长度 > 64时，此时此索引位置上的所有数据改为使用红黑树存储。

重要变量：
DEFAULT_INITIAL_CAPACITY : HashMap的默认容量，16
DEFAULT_LOAD_FACTOR：HashMap的默认加载因子：0.75
threshold：扩容的临界值，=容量*填充因子：16 * 0.75 => 12
TREEIFY_THRESHOLD：Bucket中链表长度大于该默认值，转化为红黑树:8
MIN_TREEIFY_CAPACITY：桶中的Node被树化时最小的hash表容量:64

四、LinkedHashMap的底层实现原理->源码中：

```java
static class Entry<K,V> extends HashMap.Node<K,V> {
             Entry<K,V> before, after;//能够记录添加的元素的先后顺序
             Entry(int hash, K key, V value, Node<K,V> next) {
                super(hash, key, value, next);
             }
 }
```
五、Map中定义的方法：
 添加、删除、修改操作：
 Object put(Object key,Object value)：将指定key-value添加到(或修改)当前map对象中
 void putAll(Map m):将m中的所有key-value对存放到当前map中
 Object remove(Object key)：移除指定key的key-value对，并返回value
 void clear()：清空当前map中的所有数据
 元素查询的操作：
 Object get(Object key)：获取指定key对应的value
 boolean containsKey(Object key)：是否包含指定的key
 boolean containsValue(Object value)：是否包含指定的value
 int size()：返回map中key-value对的个数
 boolean isEmpty()：判断当前map是否为空
 boolean equals(Object obj)：判断当前map和参数对象obj是否相等
 元视图操作的方法：
 Set keySet()：返回所有key构成的Set集合
 Collection values()：返回所有value构成的Collection集合
 Set entrySet()：返回所有key-value对构成的Set集合

总结：常用方法：
添加：put(Object key,Object value)
删除：remove(Object key)
修改：put(Object key,Object value)
查询：get(Object key)
长度：size()
遍历：keySet() / values() / entrySet()

TreeMap：
向TreeMap中添加key-value，要求key必须是由同一个类创建的对象
因为要按照key进行排序：自然排序 、定制排序
定制排序:

```java
TreeMap map = new TreeMap(new Comparator() {
            @Override
            public int compare(Object o1, Object o2) {
                if(o1 instanceof User && o2 instanceof User){
                    User u1 = (User)o1;
                    User u2 = (User)o2;
                    return Integer.compare(u1.getAge(),u2.getAge());
                }
                throw new RuntimeException("输入的类型不匹配！");
            }
        });
```

自然排序：

```java
public class User implements Comparable{
    ......
//按照姓名从大到小排列,年龄从小到大排列
    @Override
    public int compareTo(Object o) {
        if(o instanceof User){
            User user = (User)o;
//            return -this.name.compareTo(user.name);
            int compare = -this.name.compareTo(user.name);
            if(compare != 0){
                return compare;
            }else{
                return Integer.compare(this.age,user.age);
            }
        }else{
            throw new RuntimeException("输入的类型不匹配");
        }
}
```

Properties:
常用来处理配置文件。key和value都是String类型

## 8.Collections

Collections:操作Collection、Map的工具类

常用方法：
reverse(List)：反转 List 中元素的顺序
shuffle(List)：对 List 集合元素进行随机排序
sort(List)：根据元素的自然顺序对指定 List 集合元素按升序排序
sort(List，Comparator)：根据指定的 Comparator 产生的顺序对 List 集合元素进行排序
swap(List，int， int)：将指定 list 集合中的 i 处元素和 j 处元素进行交换
Object max(Collection)：根据元素的自然顺序，返回给定集合中的最大元素
Object max(Collection，Comparator)：根据 Comparator 指定的顺序，返回给定集合中的最大元素
Object min(Collection)
Object min(Collection，Comparator)
int frequency(Collection，Object)：返回指定集合中指定元素的出现次数
void copy(List dest,List src)：将src中的内容复制到dest中
boolean replaceAll(List list，Object oldVal，Object newVal)：使用新值替换 List 对象的所有旧值

多线程并发访问集合时的线程安全问题
Collections 类中提供了多个 synchronizedXxx() 方法，该方法可使将指定集合包装成线程同步的集合，从而可以解决。
List list1 = Collections.synchronizedList(list);  //返回的list1即为线程安全的List

## 9.比较器

通常对象之间的比较可以从两个方面去看：

第一个方面：对象的地址是否一样，也就是是否引用自同一个对象。这种方式可以直接使用“==“来完成。

第二个方面：以对象的某一个属性的角度去比较。

对于JDK8而言，有三种实现对象比较的方法：

1、覆写Object类的equals（）方法；

2、继承Comparable接口，并实现compareTo（）方法；

3、定义一个单独的对象比较器，继承自Comparator接口，实现compare（）方法。

由于使用的排序方式的不同，具体选择哪种方法来实现对象的比较也会有所不同。

第一种方法比较便于理解，复写equals()方法一般用于自己实现的对象数组排序，而对于在应用Java内置的排序算法时，使用后两种方式都是可以实现的。

先来看一下第二种方式，这种方式就是让自己编写的类继承Comparable接口，并实现接口的compareTo()方法，这种情况下，在使用java.util.Arrays.sort()方法时不用指定具体的比较器，sort()方法会使用对象自己的比较函数对对象进行排序。具体实例代码如下：

```java
import java.util.Arrays;

class BookCook implements Comparable<BookCook>{
	private String title;
	private double price;
	public BookCook(String title,double price){
		this.title = title;
		this.price = price;
	}
	@Override
	public String toString() {
		return "书名："+this.title+",价格："+this.price;
	}
	@Override
	public int compareTo(BookCook o) {
		if(this.price > o.price){
			return 1;
		}else if(this.price < o.price){
			return -1;
		}else{
			return 0;
		}
	}
}
```

从JDK1.8开始出现了Comparator接口，它的出现解决了当需要在已经开发好的代码基础上完善对象的比较功能时不想更改之前的代码的问题。这种情况，我们需要单独定义一个对象比较器，继承Comparator接口。具体实现代码如下：

```java
class Student {
	private String name;
	private double score;
	public Student(String name,double score){
		this.name = name;
		this.score = score;
	}
	public double getScore(){
		return this.score;
	}
	@Override
	public String toString() {
		return "姓名:"+this.name+",分数:"+this.score;
	}

}
class StudentComparator implements Comparator<Student> {
	@Override
	public int compare(Student o1,Student o2) {
		if(o1.getScore() > o2.getScore()){
			return 1;
		}else if(o1.getScore() < o2.getScore()){
			return -1;
		}else{
			return 0;
		}
	}
}
public class TestComparator {

	public static void main(String[] args) {

		Student[] sts = new Student[]{
				new Student("小戴",60),
				new Student("小王",90),
				new Student("老王",80),
				new Student("小萱",95)
		};

		java.util.Arrays.sort(sts, new StudentComparator());
		System.out.println(java.util.Arrays.toString(sts));
	}
}
```

# 泛型

## 泛型中的标记符含义

 E - Element (在集合中使用，因为集合中存放的是元素)

 T - Type（Java 类）

 K - Key（键）

 V - Value（值）

 N - Number（数值类型）

？ - 表示不确定的java类型

 S、U、V - 2nd、3rd、4th types

Object跟这些标记符代表的java类型有啥区别呢？ 

Object是所有类的根类，任何类的对象都可以设置给该Object引用变量，使用的时候可能需要类型强制转换，但是用使用了泛型T、E等这些标识符后，在实际用之前类型就已经确定了，不需要再进行类型强制转换。

## 泛型的使用

1.jdk 5.0新增的特性
2.在集合中使用泛型：
 总结：
 ① 集合接口或集合类在jdk5.0时都修改为带泛型的结构。
 ② 在实例化集合类时，可以指明具体的泛型类型
 ③ 指明完以后，在集合类或接口中凡是定义类或接口时，内部结构（比如：方法、构造器、属性等）使用到类的泛型的位置，都指定为实例化的泛型类型。比如：add(E e)  --->实例化以后：add(Integer e)
 ④ 注意点：泛型的类型必须是类，不能是基本数据类型。需要用到基本数据类型的位置，拿包装类替换
 ⑤ 如果实例化时，没有指明泛型的类型。默认类型为java.lang.Object类型。
3.如何自定义泛型结构：泛型类、泛型接口；泛型方法

在集合中使用泛型的情况：以ArrayList为例
ArrayList<Integer> list =  new ArrayList<Integer>();
在集合中使用泛型的情况：以HashMap为例
Map<String,Integer> map = new HashMap<String,Integer>();
jdk7新特性：类型推断
Map<String,Integer> map = new HashMap<>();
泛型的嵌套
Set<Map.Entry<String,Integer>> entry = map.entrySet();

如何自定义泛型结构：泛型类、泛型接口；泛型方法。
1.关于自定义泛型类、泛型接口：
如果定义了泛型类，实例化没有指明类的泛型，则认为此泛型类型为Object类型
要求：如果定义了类是带泛型的，建议在实例化时要指明类的泛型。

类的内部结构就可以使用类的泛型

```java
public class Order<T> {
    String orderName;
    int orderId;
    //类的内部结构就可以使用类的泛型
    T orderT;
    public Order(){
        //编译不通过
        //T[] arr = new T[10];
        //编译通过
        T[] arr = (T[]) new Object[10];
}
```

**静态方法中不能使用类的泛型。**
**异常类不能声明为泛型类**

泛型方法：在方法中出现了泛型的结构，泛型参数与类的泛型参数没有任何关系。
换句话说，泛型方法所属的类是不是泛型类都没有关系。
泛型方法，可以声明为静态的。原因：泛型参数是在调用方法时确定的。并非在实例化类时确定。

```java
public static <E>  List<E> copyFromArrayToList(E[] arr){
        ArrayList<E> list = new ArrayList<>();
        for(E e : arr){
            list.add(e);
        }
        return list;
}
```

注意：①由于子类在继承带泛型的父类时，指明了泛型类型。则实例化子类对象时，不再需要指明泛型。
②泛型不同的引用不能相互赋值。
③泛型方法在调用时，指明泛型参数的类型。

```java
泛型类的继承：
public class SubOrder extends Order<Integer> {//SubOrder:不是泛型类
    public static <E> List<E> copyFromArrayToList(E[] arr){
        ArrayList<E> list = new ArrayList<>();
        for(E e : arr){
            list.add(e);
        }
        return list;
    }
}
public class SubOrder1<T> extends Order<T> {//SubOrder1<T>:仍然是泛型类
}

泛型在继承方面的体现
①虽然类A是类B的父类，但是G<A> 和G<B>二者不具备子父类关系，二者是并列关系。
 补充：类A是类B的父类，A<G> 是 B<G> 的父类
    List<Object> list1 = null;
    List<String> list2 = new ArrayList<String>();
    //此时的list1和list2的类型不具有子父类关系
    //编译不通过
    //list1 = list2;
②通配符的使用
 通配符：?
 类A是类B的父类，G<A>和G<B>是没有关系的，二者共同的父类是：G<?>
    List<Object> list1 = null;
    List<String> list2 = null;
    List<?> list = null;
	//编译通过
    list = list1;
    list = list2;
 注意：添加(写入)：对于List<?>就不能向其内部添加数据。除了添加null之外。
      获取(读取)：允许读取数据，读取的数据类型为Object。
③有限制条件的通配符的使用。
 ? extends A:
    G<? extends A> 可以作为G<A>和G<B>的父类，其中B是A的子类
 ? super A:
    G<? super A> 可以作为G<A>和G<B>的父类，其中B是A的父类
    List<? extends Person> list1 = null;
    List<? super Person> list2 = null;

    List<Student> list3 = new ArrayList<Student>();
    List<Person> list4 = new ArrayList<Person>();
    List<Object> list5 = new ArrayList<Object>();

    list1 = list3;
    list1 = list4;
    //list1 = list5;
    //list2 = list3;
    list2 = list4;
    list2 = list5;

    //读取数据：
    list1 = list3;
    Person p = list1.get(0);
    //编译不通过
    //Student s = list1.get(0);

    list2 = list4;
    Object obj = list2.get(0);
    //编译不通过
    // Person obj = list2.get(0);

    //写入数据：
    //编译不通过
    //list1.add(new Student());
    //编译通过
    list2.add(new Person());
    list2.add(new Student());
```

# IO

## File类的使用

File类的一个对象，代表一个文件或一个文件目录(俗称：文件夹)

File类声明在java.io包下

File类中涉及到关于文件或文件目录的创建、删除、重命名、修改时间、文件大小等方法，
并未涉及到写入或读取文件内容的操作。如果需要读取或写入文件内容，必须使用IO流来完成。

后续File类的对象常会作为参数传递到流的构造器中，指明读取或写入的"终点".

1.如何创建File类的实例
    File(String filePath)
    File(String parentPath,String childPath)
    File(File parentFile,String childPath)
2.相对路径：相较于某个路径下，指明的路径。
  绝对路径：包含盘符在内的文件或文件目录的路径
3.路径分隔符
   windows:\\
   unix:/

4.在单元测试中文件目录是相对于当前module,在main方法中文件目录是相对于当前项目

常用方法：

public boolean renameTo(File dest):把文件重命名为指定的文件路径
     比如：file1.renameTo(file2)为例：
        要想保证返回true,需要file1在硬盘中是存在的，且file2不能在硬盘中存在。
public boolean isDirectory()：判断是否是文件目录
public boolean isFile() ：判断是否是文件
public boolean exists() ：判断是否存在
public boolean canRead() ：判断是否可读
public boolean canWrite() ：判断是否可写
public boolean isHidden() ：判断是否隐藏
//创建硬盘中对应的文件或文件目录
public boolean createNewFile() ：创建文件。若文件存在，则不创建，返回false
public boolean mkdir() ：创建文件目录。如果此文件目录存在，就不创建了。如果此文件目录的上层目录不存在，也不创建。
public boolean mkdirs() ：创建文件目录。如果此文件目录存在，就不创建了。如果上层文件目录不存在，一并创建
//删除磁盘中的文件或文件目录
public boolean delete()：删除文件或者文件夹
删除注意事项：要删除的文件目录下不能有子目录或文件，Java中的删除不走回收站。

public String getAbsolutePath()：获取绝对路径
public String getPath() ：获取路径
public String getName() ：获取名称
public String getParent()：获取上层文件目录路径。若无，返回null
public long length() ：获取文件长度（即：字节数）。不能获取目录的长度。
public long lastModified() ：获取最后一次的修改时间，毫秒值
如下的两个方法适用于文件目录：
public String[] list() ：获取指定目录下的所有文件或者文件目录的名称数组
public File[] listFiles() ：获取指定目录下的所有文件或者文件目录的File数组

File类提供了两个文件过滤器方法

public String[] list(FilenameFilter filter)
public File[] listFiles(FileFilter filter)

```java
File srcFile = new File("d:\\code");
File[] subFiles = srcFile.listFiles(new FilenameFilter() {
	@Override
	public boolean accept(File dir, String name) {
		return name.endsWith(".jpg");
	}
});
for(File file : subFiles){
	System.out.println(file.getAbsolutePath());
}
```

## 流

一、流的分类：
1.操作数据单位：字节流、字符流
2.数据的流向：输入流、输出流
3.流的角色：节点流、处理流

二、流的体系结构
抽象基类         节点流（或文件流）                               缓冲流（处理流的一种）
InputStream     FileInputStream   (read(byte[] buffer))        BufferedInputStream (read(byte[] buffer))
OutputStream    FileOutputStream  (write(byte[] buffer,0,len)  BufferedOutputStream (write(byte[] buffer,0,len) / flush()
Reader          FileReader (read(char[] cbuf))                 BufferedReader (read(char[] cbuf) / readLine())
Writer          FileWriter (write(char[] cbuf,0,len)           BufferedWriter (write(char[] cbuf,0,len) / flush()

```java
FileReader fr = null;
try {
	//1.实例化File类的对象，指明要操作的文件
	File file = new File("hello.txt");//相较于当前Module
	//2.提供具体的流
	fr = new FileReader(file);
	//3.数据的读入
	//read():返回读入的一个字符。如果达到文件末尾，返回-1
	//方式一：
	//int data = fr.read();
	//while(data != -1){
	//	System.out.print((char)data);
	//	data = fr.read();
	//}
	//方式二：语法上针对于方式一的修改
	int data;
	while((data = fr.read()) != -1){
		System.out.print((char)data);
	}
} catch (IOException e) {
	e.printStackTrace();
} finally {
	//4.流的关闭操作
	if(fr != null){
		try {
			fr.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
}
```

说明点：

1. read()的理解：返回读入的一个字符。如果达到文件末尾，返回-1
2. 异常的处理：为了保证流资源一定可以执行关闭操作。需要使用try-catch-finally处理
3. 读入的文件一定要存在，否则就会报FileNotFoundException。

测试FileInputStream和FileOutputStream的使用
结论：
1. 对于文本文件(.txt,.java,.c,.cpp)，使用字符流处理
2. 对于非文本文件(.jpg,.mp3,.mp4,.avi,.doc,.ppt,...)，使用字节流处理

从内存中写出数据到硬盘的文件里。
说明：

1. 输出操作，对应的File可以不存在，并不会报异常
2. File对应的硬盘中的文件如果不存在，在输出的过程中，会自动创建此文件。
   File对应的硬盘中的文件如果存在：
	如果流使用的构造器是：FileWriter(file,false) / FileWriter(file)：对原有文件的覆盖
	如果流使用的构造器是：FileWriter(file,true)：不会对原有文件覆盖，而是在原有文件基础上追加内容

```java
FileWriter fw = null;
try {
    //1.提供File类的对象，指明写出到的文件
    File file = new File("hello1.txt");
    //2.提供FileWriter的对象，用于数据的写出
    fw = new FileWriter(file,false);
    //3.写出的操作
    fw.write("I have a dream!\n");
    fw.write("you need to have a dream!");
} catch (IOException e) {
    e.printStackTrace();
} finally {
    //4.流资源的关闭
    if(fw != null){
        try {
            fw.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
//使用字符流FileReader处理文本文件
FileReader fr = null;
FileWriter fw = null;
try {
    //1.创建File类的对象，指明读入和写出的文件
    File srcFile = new File("hello.txt");
    File destFile = new File("hello2.txt");

    //不能使用字符流来处理图片等字节数据
    //            File srcFile = new File("爱情与友情.jpg");
    //            File destFile = new File("爱情与友情1.jpg");

    //2.创建输入流和输出流的对象
    fr = new FileReader(srcFile);
    fw = new FileWriter(destFile);

    //3.数据的读入和写出操作
    char[] cbuf = new char[5];
    int len;//记录每次读入到cbuf数组中的字符的个数
    while((len = fr.read(cbuf)) != -1){
        //每次写出len个字符
        fw.write(cbuf,0,len);
    }
} catch (IOException e) {
    e.printStackTrace();
} finally {
    //4.关闭流资源
    //方式一：
    //            try {
    //                if(fw != null)
    //                    fw.close();
    //            } catch (IOException e) {
    //                e.printStackTrace();
    //            }finally{
    //                try {
    //                    if(fr != null)
    //                        fr.close();
    //                } catch (IOException e) {
    //                    e.printStackTrace();
    //                }
    //            }
    //方式二：
    try {
        if(fw != null)
            fw.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    try {
        if(fr != null)
            fr.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
}

//使用字节流FileInputStream处理文本文件，可能出现乱码。
        FileInputStream fis = null;
        try {
            //1. 造文件
            File file = new File("hello.txt");
            //2.造流
            fis = new FileInputStream(file);
            //3.读数据
            byte[] buffer = new byte[5];
            int len;//记录每次读取的字节的个数
            while((len = fis.read(buffer)) != -1){
                String str = new String(buffer,0,len);
                System.out.print(str);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(fis != null){
                //4.关闭资源
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
    }
    /*
    实现对图片的复制操作
     */
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            File srcFile = new File("爱情与友情.jpg");
            File destFile = new File("爱情与友情2.jpg");

            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(destFile);

            //复制的过程
            byte[] buffer = new byte[5];
            int len;
            while((len = fis.read(buffer)) != -1){
                fos.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(fos != null){
                //
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(fis != null){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }

    //指定路径下文件的复制
    public void copyFile(String srcPath,String destPath){
        FileInputStream fis = null;
        FileOutputStream fos = null;
        try {
            File srcFile = new File(srcPath);
            File destFile = new File(destPath);

            fis = new FileInputStream(srcFile);
            fos = new FileOutputStream(destFile);
            //复制的过程
            byte[] buffer = new byte[1024];
            int len;
            while((len = fis.read(buffer)) != -1){
                fos.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(fos != null){
                //
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(fis != null){
                try {
                    fis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
    @Test
    public void testCopyFile(){
        long start = System.currentTimeMillis();
        String srcPath = "C:\\Users\\Administrator\\Desktop\\01-视频.avi";
        String destPath = "C:\\Users\\Administrator\\Desktop\\02-视频.avi";
//        String srcPath = "hello.txt";
//        String destPath = "hello3.txt";
        copyFile(srcPath,destPath);
        long end = System.currentTimeMillis();
        System.out.println("复制操作花费的时间为：" + (end - start));//618
    }
```

> 处理流之二：转换流的使用

1.转换流：属于字符流
  InputStreamReader：将一个字节的输入流转换为字符的输入流
  OutputStreamWriter：将一个字符的输出流转换为字节的输出流
  作用：提供字节流与字符流之间的转换

2.解码与编码

解码：字节、字节数组  --->字符数组、字符串
编码：字符数组、字符串 ---> 字节、字节数组

3.字符集
ASCII：美国标准信息交换码。用一个字节的7位可以表示。
ISO8859-1：拉丁码表。欧洲码表，用一个字节的8位表示。
GB2312：中国的中文编码表。最多两个字节编码所有字符
GBK：中国的中文编码表升级，融合了更多的中文文字符号。最多两个字节编码
Unicode：国际标准码，融合了目前人类使用的所有字符。为每个字符分配唯一的字符码。所有的文字都用两个字节来表示。
UTF-8：变长的编码方式，可用1-4个字节来表示一个字符。

> 标准的输入、输出流

System.in:标准的输入流，默认从键盘输入
System.out:标准的输出流，默认从控制台输出
1.1
System类的setIn(InputStream is) / setOut(PrintStream ps)方式重新指定输入和输出的流
1.2练习：
从键盘输入字符串，要求将读取到的整行字符串转成大写输出。然后继续进行输入操作，
直至当输入“e”或者“exit”时，退出程序。
方法一：使用Scanner实现，调用next()返回一个字符串
方法二：使用System.in实现。System.in  --->  转换流 ---> BufferedReader的readLine()

> 数据流

1.1 DataInputStream 和 DataOutputStream
1.2 作用：用于读取或写出基本数据类型的变量或字符串

# 锁

## 8个锁问题

1.标准访问，请问先打印邮件还是电话   //打印邮件

2.暂停4秒钟在邮件方法，请问先打印邮件还是电话  //打印邮件

3.新增普通sayHello方法，请问先打印邮件还是sayHello  //sayHello

4.两部手机，请问先打印邮件还是电话   //如何邮件方法暂停，则电话，否则邮件

5.两个静态同步方法，同一部手机，请问先打印邮件还是电话

6.两个静态同步方法，2部手机，请问先打印邮件还是电话

7.一个静态同步方法，一个普通同步方法，同一部手机，请问先打印邮件还是电话

8.一个静态同步方法，一个普通同步方法，两部手机，请问先打印邮件还是电话

```java
public class LockDemo1 {
    public static void main(String[] args) {
        Phone phone = new Phone();

        Phone phone1 = new Phone();

        new Thread(() -> {
            try {
                phone.sendEmail();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        },"A").start();

        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(() -> {
            //phone.callTel();
            phone1.callTel();
        },"B").start();

        new Thread(() -> {
            phone.sayHello();
        },"C").start();
    }
}

class Phone{
    public synchronized void sendEmail() throws InterruptedException {
        TimeUnit.SECONDS.sleep(2);
        System.out.println("send Email");
    }

    public synchronized void callTel(){
        System.out.println("call tel");
    }

    public void sayHello(){
        System.out.println("sqy hello");
    }
}
```

结论：

一个对象里面如果有多个synchronized方法，在某一个时刻，只要一个线程去调用其中的一个synchronized方法了，其他的线程都只能等待，换句话说，某一个时刻内，只能有唯一的一个线程去访问这些synchronized方法。

锁的是当前对象this，被锁定后，其他的线程都不能进入到当前对象的其他synchronized方法。

加个普通方法后发现和同步锁无关

换成两个对象后，不是同一把锁了，情况立即变化

synchronized 实现同步的基础：Java中的每一个对象都可以作为锁。具体表现为以下三种方式：

<1>对于普通同步方法，锁是当前实例对象this

<2>对于同步代码块，锁是synchronized括号里配置的对象

<3>对于静态同步方法，锁是当前类的Class对象

当一个线程试图访问同步代码块时，他必须先得到锁，退出或抛出异常时必须释放锁。也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其他非静态同步方法必须等待获取锁的方法释放锁后才能获取锁，可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁，所以无须等待该实例对象已获取锁的非静态同步方法释放锁就可以获取他们自己的锁。

所有的静态同步方法用的也是同一把锁---类对象本身，这两把锁是两个不同的对象，所以静态同步方法与非静态同步方法之间是不会有竞争条件的，但是一旦一个静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁。不管是同一个实例对象的静态同步方法之间，还是不同的实例对象的静态同步方法之间，只要他们是同一个类的实例对象，只要想获取锁，就必须等待其他静态同步方法释放锁。

## 生产者消费者问题

```java
/**
 * @author Wzl
 * @date 2020/6/23 15:57
 * @describe 题目：现在两个线程，可以操作初始值为零的一个变量，实现一个线程对该变量+1，一个线程对该变量-1，
 * 实现交替，10次，变量初始值为零
 * 1.高内聚低耦合前提下，线程操作资源类
 * 2.判断/干活/通知
 * 3.多线程下，防止虚假唤醒，进行判断时应该使用while，使用if会导致虚假唤醒
 **/
public class ProdConsumerDemo {

    public static void main(String[] args) {
        MyAge myAge = new MyAge();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myAge.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myAge.decrmrnt();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myAge.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    myAge.decrmrnt();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();
    }
}

class MyAge {

    private int number = 0;
    //As in the one argument version, interrupts and spurious wakeups are possible, and this method should always be used in a loop:
    public synchronized void increment() throws InterruptedException {
        //判断 为了防止出现虚假唤醒，这里应该使用while
        while (number != 0) {
            this.wait();
        }
        //干活
        number++;
        System.out.println(Thread.currentThread().getName() + "\t" + number);
        //通知
        this.notifyAll();
    }

    public synchronized void decrmrnt() throws InterruptedException {
        while (number == 0) {
            this.wait();
        }
        number--;
        System.out.println(Thread.currentThread().getName() + "\t" + number);
        this.notifyAll();
    }
}
```

**使用Lock改写**

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @author Wzl
 * @date 2020/6/23 15:57
 * @describe 题目：现在两个线程，可以操作初始值为零的一个变量，实现一个线程对该变量+1，一个线程对该变量-1，
 * 实现交替，10次，变量初始值为零
 * 1.高内聚低耦合前提下，线程操作资源类
 * 2.判断/干活/通知
 * 3.多线程下，防止虚假唤醒
 **/
public class ProdConsumerDemo2 {

    public static void main(String[] args) {
        MyNewAge MyNewAge = new MyNewAge();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    MyNewAge.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "A").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    MyNewAge.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "B").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    MyNewAge.increment();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "C").start();

        new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                try {
                    MyNewAge.decrement();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "D").start();

    }
}

class MyNewAge {

    private int number = 0;
    private Lock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
	//线程等待与唤醒不能再使用wait和notifyAll方法，应该使用condition的await和signalAll方法
    public void increment() throws InterruptedException {
        //干活
        lock.lock();
        try {
            //判断 为了防止出现虚假唤醒，这里应该使用while
            while (number != 0) {
                condition.await();
            }
            number++;
            System.out.println(Thread.currentThread().getName() + "\t" + number);
            //通知
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

    public void decrement() throws InterruptedException {
        lock.lock();
        try {
            while (number == 0) {
                condition.await();
            }
            number--;
            System.out.println(Thread.currentThread().getName() + "\t" + number);
            //通知
            condition.signalAll();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

> condition定点通知

```java
/**
 * @author Wzl
 * @date 2020/6/23 17:52
 * @describe 三个线程A, B, C，顺序执行，分别执行以下操作，A打印5次，B打印10次，C打印15次
 * 循环执行10轮
 **/
public class ConditionDemo2 {
    public static void main(String[] args) {
        ShareData2 sd = new ShareData2();
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                sd.Print(1);
            }, "A").start();
        }

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                sd.Print(2);
            }, "B").start();
        }

        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                sd.Print(3);
            }, "C").start();
        }
    }
}

class ShareData2 {
    private int number = 1; //A:1,B:2,C:3
    Lock lock = new ReentrantLock();
    private Condition condition1 = lock.newCondition();
    private Condition condition2 = lock.newCondition();
    private Condition condition3 = lock.newCondition();

    public void Print(int num) {
        if (num == 1) {
            lock.lock();
            try {
                if (number != 1) {
                    condition1.await();
                }
                for (int i = 0; i < 5; i++) {
                    System.out.println(Thread.currentThread().getName() + "\t" + i);
                }
                //修改标志位
                number = 2;
                condition2.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
        if (num == 2) {
            lock.lock();
            try {
                if (number != 2) {
                    condition2.await();
                }
                for (int i = 0; i < 10; i++) {
                    System.out.println(Thread.currentThread().getName() + "\t" + i);
                }
                //修改标志位
                number = 3;
                condition3.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
        if (num == 3) {
            lock.lock();
            try {
                if (number != 3) {
                    condition3.await();
                }
                for (int i = 0; i < 15; i++) {
                    System.out.println(Thread.currentThread().getName() + "\t" + i);
                }
                //修改标志位
                number = 1;
                condition1.signal();
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
}
```

## **多线程中避免发生死锁**

- 允许进程同时访问某些资源。
- 允许进程强行从占有者那里夺取某些资源。
- 进程在运行前一次性地向系统申请它所需要的全部资源。
- 把资源事先分类编号，按号分配，使进程在申请，占用资源时不会形成环路

# 其他问题

## **hashCode与equals**

hashCode相等,equals也不一定相等, 两个类也不一定相等

equals相同, 说明是同一个对象, 那么hashCode一定相同

## **访问修饰符**

（1）对于public修饰符，它具有最大的访问权限，可以访问任何一个在CLASSPATH下的类、接口、异常等。它往往用于对外的情况，也就是对象或类对外的一种接口的形式。

（2）对于protected修饰符，它主要的作用就是用来保护子类的。它的含义在于子类可以用它修饰的成员，其他的不可以，它相当于传递给子类的一种继承的东西。

（3）对于default来说，有点的时候也成为friendly（友员），它是针对本包访问而设计的，任何处于本包下的类、接口、异常等，都可以相互访问，即使是父类没有用protected修饰的成员也可以。

（4）对于private来说，它的访问权限仅限于类的内部，是一种封装的体现，例如，大多数的成员变量都是修饰符为private的，它们不希望被其他任何外部的类访问。

下表为Java访问控制符的含义和使用情况

|           | 类内部 | 本包 | 子类 | 外部包 |
| --------- | ------ | ---- | ---- | ------ |
| public    | √      | √    | √    | √      |
| protected | √      | √    | √    | ×      |
| default   | √      | √    | ×    | ×      |
| private   | √      | ×    | ×    | ×      |

区别：

（1）public：可以被所有其他类所访问。

（2）private：只能被自己访问和修改。

（3）protected：自身，子类及同一个包中类可以访问。

（4）default（默认）：同一包中的类可以访问，声明时没有加修饰符，认为是friendly。



## abstract关键字

**abstract一般用来修饰类和方法。**

1. abstract修饰类
   abstract修饰类，会使得类变成抽象类，抽象类不能生成实例，但是可以作为对象变量声明的类型，也就是编译时类型。抽象类相当于类的半成品，需要子类继承并覆盖其中的方法。
   注意：

- 抽象类虽然不能实例化，但是有自己的构造方法。
- 抽象类和接口(interface)有很大的不同之处，接口中不能有实例方法去实现业务逻辑，而抽象类可以有实例方法，并实现业务逻辑。
- 抽象类不能被final修饰，因为被final修饰的类无法被继承，而对于抽象类来说就是需要通过继承去实现抽象方法。

2. abstract修饰方法
   abstract修饰方法会使得这个方法变成抽象方法，也就是只有声明，而没有实现，需要子类重写。
   注意：

- 有抽象方法的类一定是抽象类，但是抽象类不一定有抽象方法。
- 父类是抽象类，其中有抽象方法，那么子类继承父类，并把父类中的所有抽象方法都实现了，子类才有创建对象实例的能力，否则子类也必须是抽象类。抽象类中可以有构造方法，子类在构造子类对象时需要调用父类(抽象类)的构造方法。
- 抽象方法不能用private修饰，因为抽象方法必须被子类重写，而private权限对于子类来说是不能访问的，所以就会产生矛盾。
- 抽象方法也不能用static修饰，如果用static修饰了，那么我们就可以直接通过类名调用了，而抽象方法压根没有主体，没有任何业务逻辑，这样就毫无意义了。

```java
public abstract class PublicAbstract {
    public abstract void doSthing(){ //报错：Abstract methods cannot have a body 抽象类的抽象方法不能有方法体
        
    };
}

public class PublicAbstract { //报错：Class 'PublicAbstract' must either be declared abstract or implement abstract method 'doSthing()' in 'PublicAbstract' 抽象方法不能定义在非抽象类里，但是可以定义在接口中
    public abstract void doSthing(){
    
    };
}

public abstract class PublicAbstract {
    private abstract void doSthing(); //报错：Illegal combination of modifiers: 'abstract' and 'private'
    //abstract 和 private不能同时存在于一个方法，抽象类可以定义非抽象方法
}
```

从上面的例子中可以看出：

- 抽象类是有构造方法的（当然如果我们不写，编译器会自动默认一个无参构造方法）。而且从结果来看，和普通的继承类一样，在new 一个子类对象时会优先调用父类的构造器初始化，然后再调用子类的构造器。至此相信大家都会有这样一个疑问，为什么抽象方法不能实例化却有构造器呢？ 对于这个问题网上也中说纷纭，没有确定答案。
  我是这样想的：既然它也属于继承的范畴，那么当子类创建对象时必然要优先初始化父类的属性变量和实例方法，不然子类怎么继承和调用呢？而它本身不能实例化，因为它本身就是不确定的一个对象，如果它能被实例化，那么我们通过它的对象来调用它本身的抽象方法是不是有问题。所以不能实例化有在情理之中。因此大家只要记住这个规定就行。
- 对于抽象类中的非statci(静态)和非abstract(抽象)方法中的this关键字（静态方法中不能有关键字this，因为static方法可以直接由类名调用，this指代对象，没有实例化就没有对象，所以在static方法中不使用this）代表的是它的继承类，而非抽象类本身，这个好理解，因为抽象类本身不能被实例化。如果有多个继承类，谁调用this就代表谁。

抽象类有什么好处呢？

- 由于抽象类不能被实例化，最大的好处就是通过方法的覆盖来实现多态的属性。也就是运行期绑定
- 抽象类将事物的共性的东西提取出来，由子类继承去实现，代码易扩展、易维护。

### abstract和接口的区别

1.抽象类中可以有普通成员变量，接口中没有普通成员变量。
2.抽象类和接口中都可以包含静态成员变量，抽象类中的静态成员变量的访问类型可以是任意的，但是接口中定义的变量只是public static final类型，并且默认为public static final类型。
3.抽象类可以有构造方法，接口中不能有构造方法。
4.抽象类中可以包含静态方法，接口中不能有静态方法。这里注意，静态方法不要去重写，其次这里的静态方法一定要有具体实现，不能是抽象的。Java8中允许接口中包含静态方法了，可以用接口直接调用。
5.抽象类中抽象方法的访问类型可以是public，protected，但是接口中的抽象方法只能是public类型的，并且默认为public abstract类型的。
6.一个类可以实现多个接口，但是只能继承一个抽象类。

7.接口可以继承接口。抽象类可以实现(implements)接口，抽象类可以继承具体类。抽象类中可以有静态的main方法。

- 抽象类中可以有main方法。
- 可以用抽象类来实现接口，这个时候就不需要实现接口的所有方法了。

关于Abstract的一些面试题：https://blog.csdn.net/f346867698/article/details/79435263

## final关键字

**修饰类：当用final去修饰一个类的时候，表示这个类不能被继承。**

注意：

a. 被final修饰的类，final类中的成员变量可以根据自己的实际需要设计为fianl。

b. final类中的成员方法都会被隐式的指定为final方法。说明：在自己设计一个类的时候，要想好这个类将来是否会被继承，如果可以被继承，则该类不能使用fianl修饰，在这里呢，一般来说工具类我们往往都会设计成为一个fianl类。在JDK中，被设计为final类的有String、System等

**修饰方法：被final修饰的方法不能被重写。**

注意：

a. 一个类的private方法会隐式的被指定为final方法。

b. 如果父类中有final修饰的方法，那么子类不能去重写。

**修饰成员变量**

注意：

a. 必须初始化值。

b. 被fianl修饰的成员变量赋值，有两种方式：1、直接赋值 2、全部在构造方法中赋初值。

c. 如果修饰的成员变量是基本类型，则表示这个变量的值不能改变。

d. 如果修饰的成员变量是一个引用类型，则是说这个引用的地址的值不能修改，但是这个引用所指向的对象里面的内容还是可以改变的。



## static关键字

**1 修饰代码块**

类中用static关键字修饰的代码块称为静态代码，反之没有用static关键字修饰的代码块称为实例代码块。

实例代码块会随着对象的创建而执行，即每个对象都会有自己的实例代码块，表现出来就是实例代码块的运行结果会影响当前对象的内容，并随着对象的销毁而消失(内存回收)；而静态代码块是当Java类加载到JVM内存中而执行的代码块，由于类的加载在JVM运行期间只会发生一次，所以静态代码块也只会执行一次。

因为静态代码块的主要作用是用来进行一些复杂的初始化工作，所以静态代码块跟随类存储在方法区的表现形式是静态代码块执行的结果存储在方法区，即初始化量存储在方法区并被线程共享。

**2 修饰成员变量**

类中用static关键字修饰的成员变量称为静态成员变量，因为static不能修饰局部变量(为什么？)，因此静态成员变量也能称为静态变量。静态变量跟代码块类似，在类加载到JVM内存中，JVM会把静态变量放入方法区并分配内存，也由线程共享。访问形式是：类名.静态成员名。

注意：

一个类的静态变量和该类的静态代码块的加载顺序。**类会优先加载静态变量，然后加载静态代码块，但有多个静态变量和多个代码块时，会按照编写的顺序进行加载**。

静态变量可以不用显式的初始化，JVM会默认给其相应的默认值。如基本数据类型的byte为0，short为0，char为\u0000，int为0，long为0L，float为0.0f，double为0.0d，boolean为false，引用类型统一为null。

静态变量既然是JVM内存中共享的且可以改变，那么对它的访问会引起线程安全问题(线程A改写的同时，线程B获取它的值，那么获取的是修改前的值还是修改后的值呢？)，所以使用静态变量的同时要考虑多线程情况。如果能确保静态变量不可变，那么可以用final关键字一起使用避免线程安全问题；否则需要采用同步的方式避免线程安全问题，如与volatile关键字一起使用等。

static关键不能修饰局部变量，包括实例方法和静态方法，不然就会与static关键字的初衷-共享相违背。

**3 修饰方法**

用static关键字修饰的方法称为静态方法，否则称为实例方法。通过类名.方法名调用，但需要注意静态方法可以直接调用类的静态变量和其他静态方法，不能直接调用成员变量和实例方法(除非通过对象调用)。

注意：static关键字虽然不能修饰普通类，但可以用static关键字修饰内部类使其变成静态内部类。static关键字本身的含义就是共享，而Java类加载到JVM内存的方法区，也是线程共享的，所以没必要用static关键字修饰普通类。

**static关键字的缺点**

封装是Java类的三大特性之一，也是面向对象的主要特性。因为不需要通过对象，而直接通过类就能访问类的属性和方法，这有点破坏类的封装性；所以除了Utils类，代码中应该尽量少用static关键字修饰变量和方法。

## Servlet的生命周期

1.加载：容器通过类加载器使用Servlet类对应的文件来加载Servlet

2.创建：通过**调用Servlet的构造函数来创建一个Servlet实例**

3.初始化：通过调用Servlet的init()方法来完成初始化工作，**这个方法是在Servlet已经被创建，但在向客户端提供服务之前调用。**

4.处理客户请求：Servlet创建后就可以处理请求，当有新的客户端请求时，Web容器都会**创建一个新的线程**来处理该请求。接着调用Servlet的

Service()方法来响应客户端请求（Service方***根据请求的method属性来调用doGet（）和doPost（））

5.卸载：**容器在卸载Servlet之前**需要调用destroy()方法，让Servlet释放其占用的资源。

## JSP九大内置对象和四大作用域

```xml
JSP中一共预先定义了9个内置对象：内置对象，又叫做隐含对象，不需要预先声明就可以在脚本代码和表达式中随意使用
request、response、session、application、out、pagecontext、config、page、exception

request 请求对象　 类型 javax.servlet.ServletRequest 作用域 Request
response 响应对象 类型 javax.servlet.SrvletResponse 作用域 Page
pageContext 页面上下文对象 类型 javax.servlet.jsp.PageContext 作用域 Page
session 会话对象 类型 javax.servlet.http.HttpSession 作用域 Session
application 应用程序对象 类型 javax.servlet.ServletContext 作用域 Application
out 输出对象 类型 javax.servlet.jsp.JspWriter 作用域 Page
config 配置对象 类型 javax.servlet.ServletConfig 作用域 Page
page 页面对象 类型 javax.lang.Object 作用域 Page
exception 例外对象 类型 javax.lang.Throwable 作用域 page
```



![image-20200915103142688](https://gitee.com/wzlss/tyimg/raw/master/2020/20200915103156.png)

![image-20200915103537626](https://gitee.com/wzlss/tyimg/raw/master/2020/20200915103746.png)

![image-20200915103557399](https://gitee.com/wzlss/tyimg/raw/master/2020/20200915103750.png)

## XML和JSON的区别

- XML

> 可扩展标记语言（英语：Extensible Markup Language，简称：XML）.用来传输及存储数据信息，不用来表现或展示数据。

- JSON

> - JSON 是一个轻量级的数据格式 ，不是一种编程语言。JSON是一个JavaScript 的严格的子集，利用JavaScript 中一些模式来表示结构化数据。
> - 与XML 相比，JSON是在JS中读写结构化数据更好的 方式，因为它可以把JSON 直接传给eval(),而不必创建 DOM 对象。

##### 优缺点

- XML

优点

> - 格式统一，符合标准
> - 容易与其他系统进行远程交互，数据共享比较方便

缺点

> - XML文件庞大，文件格式复杂，传输占带宽
> - 服务器端和客户端都需要花费大量代码来解析XML，导致服务器端和客户端代码变得异常复杂且不易维护
> - 客户端不同浏览器之间解析XML的方式不一致，需要重复编写很多代码
> - 服务器端和客户端解析XML花费较多的资源和时间

- JSON

优点

> - 数据格式比较简单，易于读写，格式都是压缩的，占用带宽小
>    易于解析，客户端JavaScript可以简单的通过eval()进行JSON数据的读取
> - 支持多种语言，包括ActionScript, C, C#, ColdFusion, Java, JavaScript, Perl, PHP, Python, Ruby等服务器端语言，便于服务器端的解析
> - 在PHP世界，已经有PHP-JSON和JSON-PHP出现了，偏于PHP序列化后的程序直接调用，PHP服务器端的对象、数组等能直接生成JSON格式，便于客户端的访问提取
> - 因为JSON格式能直接为服务器端代码使用，大大简化了服务器端和客户端的代码开发量，且完成任务不变，并且易于维护

缺点

> - 没有XML格式这么推广的深入人心和喜用广泛，没有XML那么通用性
> - 数据描述性不如XML

###### 数据体积方面

> JSON 相对于 XML而言，数据体积小，传递的速度比较快。

与XML序列化相比，JSON序列化后产生的数据一般要比XML序列化后数据体积小，所以在Facebook等知名网站中都采用了JSON作为数据交换方式。

###### 数据交互方面

> JSON与javascript的交互更加方便，更容易解析处理。

- 可以把 JSON 数据结构解析为有用的 JavaScript 对象。与XML数据结构要解析成 DOM 文档而且从中提取数据极为麻烦相比，JSON 可以解析为 JavaScript 对象的优势极为明显。

##### 数据描述方面

> JSON对数据的描述性比 XML 差。

- XML 文档必须包含根元素，该元素是所有其他元素的父元素。
- XML 文档的元素形成了一颗文档树，这棵树从根部开始并扩展到树的底端。
- 可读性来说，XML 文档使用简单的具有自我描述性的语法，人类语言更加贴近这种说明结构。

```xml
<?xml version="1.0" encoding="utf-8" ?>
<country>
  <name>中国</name>
  <province>
    <name>黑龙江</name>
    <citys>
      <city>哈尔滨</city>
      <city>大庆</city>
    </citys>  　　
  </province>
  <province>
    <name>广东</name>
    <citys>
      <city>广州</city>
      <city>深圳</city>
      <city>珠海</city>
    </citys> 　　
  </province>
</country>
```

- JSON 更像一个数据块，读起来比较费解，适合机器阅读。

```csharp
var country =
        {
            name: "中国",
            provinces: [
            { name: "黑龙江", citys: { city: ["哈尔滨", "大庆"]} },
            { name: "广东", citys: { city: ["广州", "深圳", "珠海"]} }
            ]
        }
```

##### 传输速度方面

> JSON 的速度要远远快于 XML。

- JSON被认为是XML的很好替代者。
- 因为JSON的可读性非常好，而且它没有像XML那样包含很多冗余的元素标签，这使得应用在使用JSON进行网络传输以及进行解析处理的速度更快，效率更高。

## 递归法

- 程序结构更简洁 
- B占用CPU的处理时间更多
- 要消耗大量的内存空间，程序执行慢，甚至无法执行