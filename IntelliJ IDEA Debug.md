### 程序调试

DEBUG 是程序开发人员必备的调试程序技能。在企业中程序开发和程序调试的比例为 1:1.5 ，是必须要掌握的技能之一。其作用主要有三个：
1. 追踪代码的运行流畅；
2. 程序运行异常定位；
3. 线上问题追踪。

### 调试说明

在程序进行调试之前，需要为指定代码设置断点（F9 快捷键）。调试过程中只涉及了 8 个功能，其对应着 8 个 IDEA 上的调试按钮：

<img src="images/IntelliJ IDEA Debug.images/image-20201128163143106.png" alt="image-20201128163143106" style="zoom:150%;" />

表格中说明了各个按钮的具体作用：

|                             按钮                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163349981.png" alt="image-20201128163349981"  /> |         Show Excution Point：快速定位到当前调试行上          |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163408266.png" alt="image-20201128163408266"  /> |    Step Over：跳过当前调试行，即不步入当前行中的方法体内     |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163426980.png" alt="image-20201128163426980"  /> | Step Into：用于步入非官方类库的方法内，但不能进入官方类库的方法中 |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163445480.png" alt="image-20201128163445480"  /> | Force Step Into：强制步入任何方法内。查看底层源码时，可以用这个功能步入官方类库的方法内 |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163502430.png" alt="image-20201128163502430"  /> |         Step Out：从当前步入的方法内退出到方法调用处         |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163518291.png" alt="image-20201128163518291"  /> |       Drop Frame：回退断点。回退到对象创建或方法调用行       |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163535679.png" alt="image-20201128163535679"  /> | Run to Cursor：运行到光标处。你可以将光标定位到指定行，使用此功能让代码运行至光标指定行，免除设置断点操作 |
| <img src="images/IntelliJ IDEA Debug.images/image-20201128163552817.png" alt="image-20201128163552817"  /> |         Evaluate Expression...：表达式计算，不常使用         |

调试过程中可以查看变量的当前值，共有三种方式：
1. 在 IntelliJ IDEA Debug.控制台内的 Variables 窗口中查看；
2. 将鼠标悬停到变量上，IDEA 会弹出相关的变量信息窗口；
3. 程序区直接查看变量。

<img src="images/IntelliJ IDEA Debug.images/image-20201128165936610.png" alt="image-20201128165936610" style="zoom:67%;" />

### 表达式计算

Evaluate Expression：表达式计算，默认快捷键 Alt + F8。

<img src="images/IntelliJ IDEA Debug.images/image-20201128163552817.png" alt="image-20201128163552817" style="zoom:200%;" />

使用计算表达式可以更改调试中变量的值：

<img src="images/IntelliJ IDEA Debug.images/image-20201128173721983.png" alt="image-20201128173721983" style="zoom:67%;" />

<img src="images/IntelliJ IDEA Debug.images/image-20201128173913361.png" alt="image-20201128173913361" style="zoom:67%;" />

<img src="images/IntelliJ IDEA Debug.images/image-20201128174009603.png" alt="image-20201128174009603" style="zoom:67%;" />

<img src="images/IntelliJ IDEA Debug.images/image-20201128174156460.png" alt="image-20201128174156460" style="zoom:67%;" />

### 条件断点

在遇到诸如 for 循环时，你可能只需要查看某个中间条件的执行结果，这个时候可以使用条件断点。

条件断点用于规定指定的条件，程序运行到符合条件时将暂停执行，此时可以从设定点开始调试或中断调试。

<img src="images/IntelliJ IDEA Debug.images/image-20201128183205232.png" alt="image-20201128183205232" style="zoom:67%;" />

### 多线程调试

启用多线程调试需要调整断点的挂起级别为Thread ，然后在 Frames 中选择线程进行调试即可。

<img src="images/IntelliJ IDEA Debug.images/image-20201128180334432.png" alt="image-20201128180334432" style="zoom:67%;" />

此时所在调试的线程是红色勾标记，而处于等待调试的线程则是红色圆点标记。

<img src="images/IntelliJ IDEA Debug.images/image-20201128180602241.png" alt="image-20201128180602241" style="zoom:67%;" />

选择 Thread-0 并运行调试至 happen 方法结束，则只有该线程会输出，其他的线程则仍会处于等待调试状态：

<img src="images/IntelliJ IDEA Debug.images/image-20201128180849873.png" alt="image-20201128180849873" style="zoom:67%;" />

如果当前线程完全结束，则会自动切换到另一个线程的调试中：

<img src="images/IntelliJ IDEA Debug.images/image-20201128181031399.png" alt="image-20201128181031399" style="zoom:67%;" />