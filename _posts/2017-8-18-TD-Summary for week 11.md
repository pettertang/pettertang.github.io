---
layout: post_layout
title: TD-Summary for week 11
time: 2017年8月18日 星期五
location: 北京
pulished: true
excerpt_separator: "　　一些常用方法如下："
---

### 第11周总结

**1**.关于HttpServletRequest  
　　HttpServletRequest接口继承自ServletRequest接口。所以先介绍下ServletRequest接口。
　　ServletRequest是一个用来向服务器提供客服端请求信息的对象。Servlet容器产生一个ServletRequest，然后将它作为参数传递给Servlet的service方法。  
　　Servlet可以提供如下数据:参数和对应值,属性,输入流。实现ServletRequest的接口可以提供另外的协议有关的数据(例如,HttpServletRequest可以提供HTTP数据)。  
　　一些常用方法如下：  
1)**String getParameter(String name)**:    
以字符串的形式返回请求参数的值,如果参数不存在,返回null, 请求参数是请求发送的而外信息, 对于HTTP servlet, 参数包含在查询字串(querystring)或发送的表格数据中。应当确定参数只有一个值的时候才使用本方法。如果参数可能有多个值,应该使用getParameterValues(Java.lang.String)
如果你对有多个值的参数使用本方法, 返回的值和getParameterValues方法返回数组的第一个值相同。
如果参数是在请求体中发送的(当使用HTTP POST方法的时候, 会出现这中情况),通过getInputStream()或getReader()直接读取body,将会干扰本方法的执行。      
2)**Enumeration getParameterNames()**:  
返回请求中所有参数名的包含字符串数据的枚举, 如果请求没有参数, 本方法返回空枚举。  
3)**String[] getParameterValues(String name)**:  
返回给定参数的所有值。如果参数不存在,返回null。如果参数只有一个值, 数组长度为1。  
4)**Map getParameterMap()**:返回请求所有参数的映射。  
5)**BufferedReader getReader()**:  
 使用BufferedReader以字符数据的形式返回the body of  request.Reader按照body总字符的编码方式对字符进行翻译。 可以使用本方法或是getInputStream()方法来读取body,但是不能够两者一块使用。  
 　　下面再列举一些HttpServletRequest中扩展的常用方法：  
 6)**StringBuffer getRequestURL()**:  
 返回客户端发出请求时的完整URL，返回的URL包含protocol, server name,port number和server path。但是不包含query string参数。如果这个请求通过RequestDispatcher.forword(javax.servlet.ServletRequest,javax.servlet.ServletResponse), 进行了转发, 重构URL的server path必须反映获取RequestDispatcher的路径, 而不是客户端确定的server path。  
 7)**String getRequestURI()**:返回请求行中的资源名部分，即在请求的首行中URL的从协议名到查询字串的部分。  
 8)**String getQueryString()**:返回请求行中的参数部分，即request URL包含的路径后面的查询字符串。  
 关于HttpServletRequest和ServletRequest还有很多常见和实用的方法，这里不过多的介绍。
 
 ---
 **2**.关于Comparable接口和Comparator接口  
 　　在项目中，有一种场景很常见，那就是需要对我们自定义的对象进行排序和比较。而这些自定义对象的比较，并不像简单的数据类型那么简单，它们往往包含了很多属性，我们通常是根据这些属性对自定义的对象进行比较。一般，Java中通过接口实现两个对象的比较，比较常用的就是Comparable接口和Comparator接口。首先类要实现接口，并且使用泛型规定要进行比较的对象所属的类，然后类实现了接口后，还需要实现接口定义的比较方法（compareTo方法或者compare方法），在这些方法中传入需要比较大小的另一个对象，通过选定的成员变量与之比较，如果大于则返回1，小于返回-1，相等返回0。  
 **2.1 Comparable接口**  
 此接口强行对实现它的每个类的对象进行整体排序。此排序被称为该类的自然排序 ，类的 compareTo方法被称为它的自然比较方法 。实现此接口的对象列表（和数组）可以通过 Collections.sort（和 Arrays.sort ）进行自动排序。实现此接口的对象可以用作有序映射表中的键或有序集合中的元素，无需指定比较器 。  
假如有如下一个例子：
```Java
class Student {
    private String name;
    private int rank;

    public Student(String _name, int _rank) {
        this.name = _name;
        this.rank = _rank;
    }
    public String toString() {
        return this.name + ":" + this.rank;
    }
}
```
我们有这样一个自定义的Student类，类中有名字和排名两个属性。如果我们想要对Student类的对象进行排序操作，像下面这样：
```Java
public class ComparableTest {
    public static void main(String[] args) {
        List<Student> list = new ArrayList<>();
        list.add(new Student("Tom",4));
        list.add(new Student("Jack",8));
        list.add(new Student("Tony",6));
        System.out.println(list);
        Collections.sort(list);       //编译报错
        System.out.println(list);
    }
}
```
我们是无法直接对这样的Student对象直接排序的，其中一种常见实现方式是让Student类实现Comparable接口，如下所示：
```Java
class Student implements Comparable<Student>{
    private String name;
    private int rank;

    public Student(String _name, int _rank) {
        this.name = _name;
        this.rank = _rank;
    }
    public String toString() {
        return this.name + ":" + this.rank;
    }

    @Override
    public int compareTo(Student o) {
        return this.rank - o.rank;
    }
}
```
我们可以看到，Student类实现了Comparable接口并重写了compareTo方法。我们例子中重写的compareTo方法，根据Student的rank成员属性来进行比较，就和name属性没有关系了。然后刚才上面有报错的ComparableTest类就不会再报错了，里面的main方法会成功执行，执行结果如下：
```Java
[Tom:4, Jack:8, Tony:6]
[Tom:4, Tony:6, Jack:8]
```
**2.2 Comparator接口**  
Comparator接口与上面的Comparable接口不同的地方是：  
1）Comparator位于包java.util下，而Comparable位于包java.lang下。  
2）Comparable接口将比较代码嵌入需要进行比较的类的自身代码中，而Comparator接口在一个独立的类中实现比较。  
3）如果前期类的设计没有考虑到类的Compare问题而没有实现Comparable接口，后期可以通过Comparator接口来实现比较算法进行排序，并且为了使用不同的排序标准做准备，比如：升序、降序。  
4）Comparable接口强制进行自然排序，而Comparator接口不强制进行自然排序，可以指定排序顺序。  
我们还是用最开始的Student类的例子来做演示：
```Java
class Student1 {
    private String name;
    private int rank;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getRank() {
        return rank;
    }

    public void setRank(int rank) {
        this.rank = rank;
    }
    public Student1(String _name, int _rank) {
        this.name = _name;
        this.rank = _rank;
    }
    public String toString() {
        return this.name + ":" + this.rank;
    }
}
```
然后我们实现比较器类如下：
```Java
class StudentCompare implements Comparator<Student1> {

    @Override
    public int compare(Student1 o1, Student1 o2) {
        if (o1.getRank() > o2.getRank()) {
            return 1;
        }
        if (o1.getRank() < o2.getRank()) {
            return -1;
        }
        return 0;
    }
}
```
相应的Test测试类如下：
```Java
public class ComparatorTest {
    public static void main(String[] args) {
        List<Student1> list = new ArrayList<>();
        list.add(new Student1("Tom",4));
        list.add(new Student1("Jack",8));
        list.add(new Student1("Tony",6));
        System.out.println(list);
        StudentCompare studentCompare = new StudentCompare();
        Collections.sort(list,studentCompare);
        System.out.println(list);
    }
}
```
同样，我们可以得到和2.1中一样的输出结果。
```Java
[Tom:4, Jack:8, Tony:6]
[Tom:4, Tony:6, Jack:8]
```

 
 
