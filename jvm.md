#Java Virtual Machine

##类加载（load class）
[摘自:from](http://www.importnew.com/18548.html)
### 过程
1. **加载(Loading)**
    
    1. 先读取流(文件/网络/..)内容到内存。
    2. 生成对应的静态结构（方法结构）到静态区
    3. 生成对应类的Class对象
    
    ps:加载和连接(验证阶段)可能交叉，不过肯定是开始时间是顺序的
2. **验证(Verification):(此过程为连接Linking内的一个环节)**

    验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所引用的类经过反复验证，那么可以考虑采用-Xverifynone参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。
    
    1. **文件格式验证:** 验证字节流是否符合Class文件格式的规范；例如：是否以魔术0xCAFEBABE开头、主次版本号是否在当前虚拟机的处理范围之内、常量池中的常量是否有不被支持的类型。
    2. **元数据验证:** 对字节码描述的信息进行语义分析（注意：对比javac编译阶段的语义分析），以保证其描述的信息符合Java语言规范的要求；例如：这个类是否有父类，除了java.lang.Object之外。
    3. **字节码验证:** 通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的。
    4. **符号引用验证:** 确保解析动作能正确执行。
    
    
3. **准备(Preparation):(此过程为连接Linking内的一个环节)**

    给类的静态属性分配引用内存并且给他系统的默认值，如果是final会直接构造
    
4. **解析(Resolution):(此过程为连接Linking内的一个环节)**

    解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号引用进行。

5. **初始化(Initialization)**

    <cinit()>:静态代码初始化
    <init()>:实例初始化
    这是类加载的最后一步，他会初始化所有的静态属性并且调用静态代码块，也就是<cinit()>，静态属性和静态代码块的执行顺序按照定义的顺序执行，静态代码块可以改写在其后的属性，但是不能读取。
    有父子继承关系的类会先执行完父类的<cinit()>再执行子类的<cinit()>,接口也有<cinit()>。<cinit()>是阻塞的，调用的只能为一个线程，如果有多个线程调用则会等待，如果有大延迟操作那是致命的。
    一下情况会触发静态初始化：
    1. 遇到new,getstatic,putstatic,invokestatic这失调字节码指令时，如果类没有进行过初始化，则需要先触发其初始化。生成这4条指令的最常见的Java代码场景是：使用new关键字实例化对象的时候、读取或设置一个类的静态字段（被final修饰、已在编译器把结果放入常量池的静态字段除外）的时候，以及调用一个类的静态方法的时候。
    1. 使用java.lang.reflect包的方法对类进行反射调用的时候，如果类没有进行过初始化，则需要先触发其初始化。
    1. 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
    1. 当虚拟机启动时，用户需要指定一个要执行的主类（包含main()方法的那个类），虚拟机会先初始化这个主类。
    1. 当使用jdk1.7动态语言支持时，如果一个java.lang.invoke.MethodHandle实例最后的解析结果REF_getstatic,REF_putstatic,REF_invokeStatic的方法句柄，并且这个方法句柄所对应的类没有进行初始化，则需要先出触发其初始化。
    
    
6. **使用(Using)**
7. **卸载(Unloading)**

加载、验证、准备、初始化和卸载这5个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班地开始，而解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定（也称为动态绑定或晚期绑定）。以上描述的是Hotspot的加载机制。

































































