## JVM 的参数类型

### 标准参数（各版本中保持稳定）

- -help

- -server -client

- -version -showversion

- -cp -classpath

### X 参数（非标准化参数）

- -Xint：解释执行

- -Xcomp：第一次使用就编译成本地代码

- -Xmixed：混合模式，JVM 自己决定是否编译成本地代码

示例：

```java
java -version（默认是混合模式）
Java HotSpot(TM) 64-Bit Server VM (build 25.40-b25, mixed mode)

java -Xint -version
Java HotSpot(TM) 64-Bit Server VM (build 25.40-b25, interpreted mode)
```

### XX 参数（非标准化参数）

主要用于 JVM调优和 debug

- Boolean类型

```java
格式：-XX:[+-]<name>表示启用或禁用 name 属性
如：-XX:+UseConcMarkSweepGC
-XX:+UseG1GC
```

- 非Boolean类型

```java
格式：-XX:<name>=<value>表示 name 属性的值是 value
如：-XX:MaxGCPauseMillis=500
-xx:GCTimeRatio=19
-Xmx -Xms属于 XX 参数
-Xms 等价于-XX:InitialHeapSize
-Xmx 等价于-XX:MaxHeapSize
-xss 等价于-XX:ThreadStackSize
```

## JVM 的参数查看

### 查看指定参数

```shell
> jinfo -flag MaxHeapSize <pid>
-XX:InitialHeapSize=134217728
```

### 查看所有参数

```shell
> jinfo -flags <pid>
Attaching to process ID 736, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 25.171-b11
Non-default VM flags: -XX:CICompilerCount=3 -XX:InitialHeapSize=134217728 -XX:MaxHeapSize=2128609280 -XX:MaxNewSize=709361664 -XX:MinHeapDeltaBytes=524288 -XX:NewSize=44564480 -XX:
OldSize=89653248 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
Command line:  -ea -Didea.test.cyclic.buffer.size=1048576 -javaagent:C:\Program Files\JetBrains\IntelliJ IDEA 2019.2.4\lib\idea_rt.jar=62900:C:\Program Files\JetBrains\IntelliJ IDE
A 2019.2.4\bin -Dfile.encoding=UTF-8
```

### 查看所有JVM参数启动时的初始值

```shell
> java -XX:+PrintFlagsInitial
```

### 查看所有JVM参数运行时生效的值

```shell
-XX:+PrintFlagsFinal
```

### 其他

```
-XX:+UnlockExperimentalVMOptions 解锁实验参数

-XX:+UnlockDiagnosticVMOptions 解锁诊断参数

-XX:+PrintCommandLineFlags 打印命令行参数
```

输出结果中=表示默认值，**:=表示被用户或 JVM 修改后的值**

>  pid 可通过类似 ps -ef|grep java/tomcat或 jps来进行查看

