## java8新特性

* 接口静态方法
* 接口默认方法
* lambda
* stream
* LocalDate
* ...

### 接口静态方法
	interface A{
	static void a{}
	}
##### ps:只允许为public，不写默认public

### 接口默认方法
	interface A{
	default void a{}
	}
##### ps:多实现的时候如果都含有同一个函数则编译失败

### MR Method Reference 方法引用过

方法引用的3种写法
* objectName::instanceMethod
* ClassName::staticMethod
* ClassName::instanceMethod ps:这个引用的方法为ClassName::instanceMethod(this,param),类似于生成了一个静态方法instanceMethod，然后实现为ClassA:instanceMethod((ClassA)thiz,param)->{thiz.instanceMethod(param)}
##### ps：其实方法参数是这样的lambda Math::max等同于(x, y)->Math.max(x,y)

补充上面的方法引用，官方给出一个排序的实例：

	    public class Person{
	        private final String firstName;
	        private final String lastName;
	        private final int age;
    
	        public Person(String fn, String ln, int a) {
		        this.firstName = fn; this.lastName = ln; this.age = a;
	        }

	        public String getFirstName() { return firstName; }
	        public String getLastName() { return lastName; }
	        public int getAge() { return age; }
	    }
		
	    arr.stream().sorted(Comparator.comparing(Person::getFirstName));//arr 是Person的List
##### ps1: Comparator这个接口下的comparing这个默认方法可以把一个无参,返回值实现了Comparator的类方法引用（上述第三种）按照返回值的comparable生成comparator，参数写出来形式就是> T extends Comparable instanceMethod()的Class::instanceMethod引用。
##### ps2: 笔者是根据这个方法分析出第三种方法其实是会生成一个带this的参数的结论的。
##### ps3: 看comparing源码还发现了个有意思的东西是java8允许多转换比如class A implements Interface1,Interface2.则可以写成 (Interface1&Interface2)instance的强转形式，意味着instance这个类必须实现了Interface1并且实现了Interface2，上述的类A的实例a就可以写成Interface2 i2=(Interface1&Interface2)a;Interface1 i1=(Interface1&Interface2)a;可以参照comparing的源码
##### ps4: 笔者以为lambda不会自动装箱的，不过从上面的实例发现会自动装箱（Integer.valueOf(getAge())）

### lambda

所有的单函数未实现接口都可以写成lambda(只能是接口，虚类无效)，下面是lambda的书写
参数列表 -> 函数体
无参数写成()->{}
单参数，单函数体语句写成 a->System.out.println(a)，单语句的时候无需写return，lambda会自动判断
函数体的this为调用类的实例

@FunctionalInterface可以标记一个接口为函数式接口，实际上不标记只要符合只有一个为实现的接口就可以写成lambda
#### 常用函数接口
* java.util.function.BiConsumer //2入0出
* java.util.function.BiFunction //2入1出
* java.util.function.BiPredicate //2元判断
* java.util.function.Consumer //1入0出
* java.util.function.Function //1入1出
* java.util.function.Predicate //1元判断
* java.util.function.Supplier //0入1出
* java.util.function.UnaryOperator //继承Function，多个identity()函数，可以获得一个一元函数，进=出

### stream
流被操作或者关闭就不可再次使用。
对流设置的所有操作都会在reduce回归的时候被一次性执行，并且会遍历整个流。可以看做你设置流的所有操作会生成一个操作链，然后再回归的时候遍历整个流，然后每个元素都进过一个操作链进行处理。


#### 生产

* Collection#stream(); //用当前collection的内容生成个stream
* Stream.generate(supplier) //使用supplier生成无限长的stream
* Stream.concat(stream1,stream2) //用两个流流入一个新的流
* Stream.empty() //生成一个空流
* Stream.of(t) //生成一个只有t的流
* Stream.of(t1,t2...) //生成一个新流，把参数流进去
* Stream.iterate(1,function) //迭代成一个新的流，参数2的返回值会放入流中，并且把返回值传递给下一次函数调用

#### 转换

* arr.stream().distinct() //去重，用老流生成一个没重复的新流
* arr.stream().filter(predicate) //过滤 用老流生成一个predicate判断为true的所有元素的新流 断言为true的留下来
* arr.stream().map(Function) //转型 用老流生成一个所有元素经过一元函数（Function）转型的新流 
* arr.stream().mapToInt(Function) //转换成IntStream里面实现了一些int类型的回归方法，比如max就不用自己实现comparator了
* arr.stream().mapToLong(Function) //转换成LongStream里面实现了一些long类型的回归方法，比如max就不用自己实现comparator了
* arr.stream().mapToDouble(Function) //转换成DoubleStream里面实现了一些double类型的回归方法，比如max就不用自己实现comparator了
* arr.stream().flatMap(Function<t,stream>) //转型(一元流转换函数) 生成一个新流(S流)，老流的所有元素经过一元函数（Function<t,stream>)转换成一个流，并依次把内容流入开始生成的流(S流)里    例子：

	Arrays.asList(Arrays.asList(1,2,3,4,5),Arrays.asList(6,7,8,9,0)).stream().flatMap(list->list.stream()).forEach(i->System.out.print(i));

* arr.stream().peek(Consumer) //设置回调 用老流生成一个新流，并且接下来的遍历流的元素的时候每次都会调用一下回调(consumer)
* arr.stream().limit(n) //截断流 用老流生成一个新流，且只取前n个，没那么长则全取
* arr.stream().skip(n) //略过前n个，用老流生成一个新流，且略过前n个，没那么长则空流
* arr.stream().sorted([comparator]) //用老流生成一个排序了的新流

#### 回归（结流）

* arr.stream().collect(Collector) //收集，Collectors下定义的常量,也可以自定义Collector
* arr.stream().collect(supplier,bconsumer1,bconsumer2) //收集，第一个是生产器，要生产一个用于返回的结果，第二个参数是添加器用于把流内容加入到返回结果，第三个参数是把另一个结果集的内容加入到此结果集，例子如下
	
	LinkedList<Integer> llist=arr.stream().collect(
	()->new LinkedList<Integer>(),
	(list,item)->list.add(item),
	(list1,list2)->list1.addAll(list2)
	);

* arr.stream().reduce(BinaryOperator) //回归，两元一次函数,第一个参数是上次堆叠的结果（如果是第一次则为流的第一个元素，从第二个元素开始堆叠），第二个是这次堆叠的流元素，返回值为下一个堆叠的第一个参数
* arr.stream().reduce(init,BinaryOperator) //类似于↑，不过堆叠内容从init开始，返回值直接为init的类型
* arr.stream().count() //计算个数
* arr.stream().max(comparator) //找到最大的一个
* arr.stream().findAny() //找到任意一个
* arr.stream().findFirst() //找到第一个
* arr.stream().allMatch(predicate) //是否流内所有元素都满足predicate
* arr.stream().anyMatch(predicate) //是否流内任意元素满足predicate

##### ps:stream的回归一直从最开始进行遍历比如:Arrays.asList(1,2,3,4,5).stream().peek(i->System.out.print("["+i+"]")).skip(2).limit(1).forEach(i-> System.out.print(i+" "));输出为[1][2][3]3 ，可以从下图看流程：
	{peek->skip}      {peek&forEach}   {limit trash rest}
	 1        2             3          4 5
	[1]      [2]          [3]3
###### java.util.Optional
回归的时候有很多参数都是Optional类型的，这个类型是java为了防止空而做的处理，用get获取值，isPresent来判空














