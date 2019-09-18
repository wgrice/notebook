从 JDK 1.6 开始引入了用 Java 代码重写的编译器接口，使得我们可以在运行时编译 Java 源码，然后用类加载器进行加载，让 Java 语言更具灵活性，能够完成许多高级的操作。

### 从源文件到字节码文件的编译方式

对于一个 java 源文件

```java
// Example.java
public class Example{
  @Override
  public String toString() {
    return "hello java compiler";
  }
}
```

传统的编译方式是使用命令行在当前目录下运行

```shell
javac Example.java
```

然后在同一目录下生成 Example.class 字节码文件。而使用 Java API 来编译类文件则稍微有点复杂，示例代码如下

```java
public class CompileJavaFile {

    public static void main(String[] args) {
        // 获取系统Java编译器
        JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();
        // 获取Java文件管理器
        StandardJavaFileManager fileManager = compiler.getStandardFileManager(null, null, null);
        // 定义要编译的源文件
        File file = new File("path/to/file");
        // 通过源文件获取到要编译的 Java 类源码迭代器，包括所有内部类，其中每个类都是一个 JavaFileObject，也被称为一个汇编单元
        Iterable<? extends JavaFileObject> compilationUnits = fileManager.getJavaFileObjects(file);
        // 生成编译任务
        JavaCompiler.CompilationTask task = compiler.getTask(null, fileManager, null, null, null, compilationUnits);
        // 执行编译任务
        task.call();
    }

}
```

上述代码运行之后，会在 Example.java 的同一目录下生成 Example.class 文件，之所以这么简单的任务还要写这么长的代码，还是与 Java 的设计风格分不开。比如说第一句获取到的编译器实例，在这里获得的是系统使用的 Java 编译器，但 API 的设计者却想让广大的程序员有机会定制有自己风格的编译器，于是使用了一个接口来定义必要的通用行为，同样的思想还用在了后续几个类的设计上面。

#### task.call() 的调用流程

为了说明上述代码的工作原理，我们来捋一捋整个过程的重要函数调用，以及数据传递链。首先从 task.call() 这一句开始，直接点进去发现它是 JavaCompiler 接口的内部接口 CompilationTask 的接口方法。而 CompilationTask 接口的直接子类是 JavacTask，这是一个抽象类，没有实现 call() 方法，继续寻找子类发现了 BaseJavacTask，它的 call() 方法直接抛出异常，所以这个类不能调用该方法，最后在子类 JavacTaskImpl 中找到了该方法实现

```java
@Override @DefinedBy(Api.COMPILER)
public Boolean call() {
    return doCall().isOK();
}

/* Internal version of call exposing Main.Result. */
public Main.Result doCall() {
    try {
        return handleExceptions(() -> {
            prepareCompiler(false);
            if (compiler.errorCount() > 0)
                return Main.Result.ERROR;
            compiler.compile(args.getFileObjects(), args.getClassNames(), processors, addModules);
            return (compiler.errorCount() > 0) ? Main.Result.ERROR : Main.Result.OK; // FIXME?
        }, Main.Result.SYSERR, Main.Result.ABNORMAL);
    } finally {
        try {
            cleanup();
        } catch (ClientCodeException e) {
            throw new RuntimeException(e.getCause());
        }
    }
}
```

除去外围的异常处理，这其中最重要的一句是编译器的编译方法调用，传入了要编译的文件对象集合 args.getFileObjects()，以及类名称集合 args.getClassNames()。

```java
compiler.compile(args.getFileObjects(), args.getClassNames(), processors, addModules);
```

进入 compile() 方法，这里面有很多内容，根据对前面代码的单步调试过程，我们定位到了字节码生成的方法调用

```java
case BY_TODO:
	while (!todo.isEmpty())
		generate(desugar(flow(attribute(todo.remove()))));
	break;
```

而在 generate() 方法中，关键代码是

```java
JavaFileObject file;
...
...
file = genCode(env, cdef);
```

这就是将生成的字节码内容存储到一个 JavaFileObject 实例中，我们进入到 genCode() 方法发现了

```java
if (gen.genClass(env, cdef) && (errorCount() == 0))
    return writer.writeClass(cdef.sym);
```

具体的生成方法先不管，这涉及到 Java 语言编译的内容，我们看第二行，它表示使用一个 ClassWriter 写入字节码内容，并返回一个 JavaFileObject 实例，它的参数是一个 ClassSymbol 类，里面有两个 JavaFileObject，一个是 sourceFile ，表示源码内容，另一个是 classFile， 表示生成的字节码内容。在 writeClass 方法中，JavaFileObject 对象的创建方法为

```java
JavaFileObject outFile
            = fileManager.getJavaFileForOutput(outLocn,
                                               name,
                                               JavaFileObject.Kind.CLASS,
                                               c.sourcefile);
```

之所以叫 outFile 是因为这是一个包含有编译后字节码内容的 Java 文件对象，是需要被输出的，JavaFileObject.Kind.CLASS 表明这是一个包含字节码的文件对象。然后紧接着打开输出流

```java
OutputStream out = outFile.openOutputStream();
```

并使用下面的函数写入到文件

```java
writeClassFile(out, c);
```

上面的代码跟踪大致提取出了 Java 源码的编译执行流程

![img](C:\wg\project\git\notebook\image\func-call.png)

#### 输出流跟踪

前面对 task.call() 方法的分析，只梳理了一下骨架，却没有涉及其中的必要细节，比如最后一步向文件写入字节码的时候，就不知道具体使用的是 OutputStream 的哪个实现类，下面我们进一步分析。在 ClassWriter 类的 writeClass() 方法中，OutputStream 的获取代码为

```java
OutputStream out = outFile.openOutputStream();
```

其中 outFile 是一个 JavaFileObject 实例，于是这个 OutputStream 的实际类型就依赖于特定的Java文件对象的 openOutputStream() 方法实现，而 outFile 又是通过一个 FileManager 实例生成的。在 ClassWriter 类中，FileManager 实例出现在构造方法中

```java
fileManager = context.get(JavaFileManager.class);
```

找到它的调用处，原来是在它的静态方法 ClassWriter.instance() 中，而 instance() 被 package com.sun.tools.javac.main 包里面的 JavaCompiler 的构造方法调用。再寻找 main/JavaCompiler 的构造方法调用， 原来也在其静态方法 JavaCompiler.instance() 中被调用，而 JavaCompiler.instance() 方法则被 JavacTaskImpl 中的私有方法 prepareCompiler() 调用，从名称可知这是一个编译前的准备工作，再上一层，我们来到了熟悉的 doCall() 方法。

我们将上面的一连串方法调用用下图表示出来

![img](C:\wg\project\git\notebook\image\func-call-filemanager.png)

可以看到，FileManager 实例最终是从 Context 对象中获取的，然后再来对 Context 对象进行跟踪。就目前为止，Context 实例最早出现于 JavacTaskImpl 类的构造方法，作为构造参数被传入。而 JavacTaskImpl 的构造出现在 JavacTool 的 getTask() 方法中

```java
@Override @DefinedBy(Api.COMPILER)
public JavacTask getTask(Writer out,
                             JavaFileManager fileManager,
                             DiagnosticListener<? super JavaFileObject> diagnosticListener,
                             Iterable<String> options,
                             Iterable<String> classes,
                             Iterable<? extends JavaFileObject> compilationUnits) {
    Context context = new Context();
    return getTask(out, fileManager, diagnosticListener,
                options, classes, compilationUnits,
                context);
}

  /* Internal version of getTask, allowing context to be provided. */
public JavacTask getTask(Writer out,
                             JavaFileManager fileManager,
                             DiagnosticListener<? super JavaFileObject> diagnosticListener,
                             Iterable<String> options,
                             Iterable<String> classes,
                             Iterable<? extends JavaFileObject> compilationUnits,
                             Context context)
{
        try {
            ClientCodeWrapper ccw = ClientCodeWrapper.instance(context);

            if (options != null) {
                ...
            }

            if (classes != null) {
                ...
            }

            if (compilationUnits != null) {
                ...
            }

            if (diagnosticListener != null)
                ...

            if (out == null)
                ...
            else
                ...

            if (fileManager == null) {
                fileManager = getStandardFileManager(diagnosticListener, null, null);
                if (fileManager instanceof BaseFileManager) {
                    ((BaseFileManager) fileManager).autoClose = true;
                }
            }
            fileManager = ccw.wrap(fileManager);

            context.put(JavaFileManager.class, fileManager);

            return new JavacTaskImpl(context);
        } catch (PropagatedException ex) {
            throw ex.getCause();
        } catch (ClientCodeException ex) {
            throw new RuntimeException(ex.getCause());
        }
    }
```

getTask() 有两个重载方法，第一个方法新建了一个 Context 对象，然后调用第二个 getTask()，并传入自身所有参数以及 Context 对象。而实际起作用的 getTask() 方法，则主要是在处理传入的各种参数，并组装 Context 对象，其中包括将 fileManager 存入 context。而这里的 fileManager 就是我们在客户端代码中传入的 StandardJavaFileManager 实例。

于是我们看到，在 ClassWriter 类中，JavaFileObject 是从我们最初传入的文件管理器 fileManager 中获取的。进入 StandardJavaFileManager 发现这只是一个继承了 FileManager 的子接口，于是再从 javax.tool 包里面的 JavaCompiler 入手，发现 getStandardFileManager() 只是一个接口方法，它的实现在 JavacTool 类中

```java
@Override @DefinedBy(Api.COMPILER)
    public JavacFileManager getStandardFileManager(
        DiagnosticListener<? super JavaFileObject> diagnosticListener,
        Locale locale,
        Charset charset) {
        Context context = new Context();
        context.put(Locale.class, locale);
        if (diagnosticListener != null)
            context.put(DiagnosticListener.class, diagnosticListener);
        PrintWriter pw = (charset == null)
                ? new PrintWriter(System.err, true)
                : new PrintWriter(new OutputStreamWriter(System.err, charset), true);
        context.put(Log.errKey, pw);
        CacheFSInfo.preRegister(context);
        return new JavacFileManager(context, true, charset);
    }
```

也就是说从 getStandardFileManager() 方法中得到的文件管理器是一个 JavacFileManager 实例，进入它的 getJavaFileForOutput() 方法，我们看到一个重载方法

```java
public JavaFileObject getJavaFileForOutput(Location location,
                                             String className,
                                             JavaFileObject.Kind kind,
                                             FileObject sibling)
```

该方法调用另一个私有重载方法

```java
private JavaFileObject getFileForOutput(Location location,
                                        RelativeFile fileName,
                                        FileObject sibling)
```

里面有多个返回语句，通过断点调试，我们定位到了这一句

```java
 return ((PathFileObject) sibling).getSibling(baseName);
```

也就是说，具体的返回类型依赖于 sibling，而这个 sibling 是作为 getJavaFileForOutput 参数传入的

```java
JavaFileObject outFile
            = fileManager.getJavaFileForOutput(outLocn,
                                               name,
                                               JavaFileObject.Kind.CLASS,
                                               c.sourcefile);
```

具体来讲，sibling 就是上面的 c.sourceFile ，之所以这个变量叫 sibling ，就是因为 outFile 是 c.sourceFile 对应的字节码文件对象。从上面的代码来看，返回的Java 文件对象是一个 PathFileObject 实例，而 PathFileObject 本身是一个抽象类，它有四个子类，分别是 JarFileObject、DirectoryFileObject、SimpleFileObject、JRTFileObject。从名称来看应该表示的是四种文件对象源码获取路径，即Jar包、文件夹、文件以及jrt文件系统。而在我们的例子中，应该使用的是 SimpleFileObject 。在 PathFileObject 中我们找到了 openOutputStream 实现

```java
@Override @DefinedBy(Api.COMPILER)
public OutputStream openOutputStream() throws IOException {
        fileManager.updateLastUsedTime();
        fileManager.flushCache(this);
        ensureParentDirectoriesExist();
        return Files.newOutputStream(path);
}
```

最终我们知道了，原来 ClassWriter 中使用的输出流来自于 nio 包的 Files 类，于是将字节码写入文件就合情合理了。

### 从源文件到内存的编译方式

从前面的分析我们看到，JavaFileObject 的 openOutputStream() 方法控制了编译后字节码的输出行为，也就意味着我们可以根据需要定制自己的 Java 文件对象。比如，当编译完源文件之后，我们不想将字节码输出到文件，而是留在内存中以便后续加载，那么我们可以实现自己的输出文件类 JavaFileObject。由于输出文件对象是从文件管理器的 getJavaFileForOutput() 方法获取的，所以我们还应该重写文件管理器的这一行为，综合起来的代码如下

```java
JavaFileManager jfm = new ForwardingJavaFileManager(fileManager) {
            public JavaFileObject getJavaFileForOutput(JavaFileManager.Location location,
                                                       String className,
                                                       JavaFileObject.Kind kind,
                                                       FileObject sibling) throws IOException {
                if(kind == JavaFileObject.Kind.CLASS) {
                    return new SimpleJavaFileObject(URI.create(className + ".class"), JavaFileObject.Kind.CLASS) {
                        public OutputStream openOutputStream() {
                            return new FilterOutputStream(new ByteArrayOutputStream()) {
                                public void close() throws IOException{
                                    out.close();
                                    ByteArrayOutputStream bos = (ByteArrayOutputStream) out;
                                    bytes.put(className, bos.toByteArray());
                                }
                            };
                        }
                    };
                }else{
                    return super.getJavaFileForOutput(location, className, kind, sibling);
                }
            }
        };
```

其中 bytes 就是存储字节码的容器，它的键为类名，值为类的字节码数组。这里的 SimpleJavaFileObject 是 JavaFileObject 的子类，实现了大多数必要方法，我们只需要复写 openOutputStream() 即可。

然后将 jfm 传入 getTask() 方法获取任务并执行

```java
JavaCompiler.CompilationTask task = compiler.getTask(null, jfm, null, null, null, compilationUnits);
task.call();
```

当要使用编译后的类时，直接使用类加载器加载 bytes 中的字节码即可。

### 从内存到内存的编译方式

既然能够自定义 JavaFileObject 来控制字节码的输出行为，那么很明显也能够通过类似的方式来控制源码的输入，比如从字符串中读取源码而非从文件中。首先观查下面的代码片段（这是从最初的代码中截取的）

```java
File file = new File("/path/to/file");
//通过源文件获取到要编译的Java类源码迭代器，包括所有内部类，其中每个类都是一个 JavaFileObject，也被称为一个汇编单元
Iterable<? extends JavaFileObject> compilationUnits = fileManager.getJavaFileObjects(file);
//生成编译任务
JavaCompiler.CompilationTask task = compiler.getTask(null, fileManager, null, null, null, compilationUnits);
```

与字节码输出 JavaFileObject 对象有点不一样的是，表示源码的 JavaFileObject 是作为参数传入 getTask() 方法的，这时候的 fileManager 更像是一个工具类用于把 File 对象数组自动转换成 JavaFileObject 列表，换成手动生成 compilationUnits 列表并传入也是可行的。所以要修改源码输入方式，用不着复写文件管理器类。

为了获取源码读入的关键代码，我们再进行一次调用追踪，这次应该要轻松一点。仍然首先从 call() 开始，定位到 JavacTaskImpl 的 doCall() 方法，进入 main/JavaCompiler 的 compile() 方法，通过跟踪参数 sourceFileObjects，我们注意到

```java
// These method calls must be chained to avoid memory leaks
processAnnotations(
  enterTrees(
    stopIfError(CompileState.PARSE,
              initModules(stopIfError(CompileState.PARSE, parseFiles(sourceFileObjects))))
  ),
  classnames
);
```

其中的 parseFiles(sourceFileObjects) 显然是对源文件的解析，进入该方法，找到了

```java
trees.append(parse(fileObject));
```

再进入 parse() 方法，很简单的几句，轻松找到

```
 JCTree.JCCompilationUnit t = parse(filename, readSource(filename));
```

我们很欣喜地看到 readSource(filename)

```
public CharSequence readSource(JavaFileObject filename) {
    try {
        inputFiles.add(filename);
        return filename.getCharContent(false);
    } catch (IOException e) {
        log.error("error.reading.file", filename, JavacFileManager.getMessage(e));
        return null;
    }
}
```

也就是说源码的读取是通过 JavaFileObject 的 getCharContent() 方法。在 PathFileObject 的 getCharContent 内部，返回的是 CharBuffer 实例，因为是从文件读取，所以代码还是有几行，但是如果源码是字符串形式的，那么直接使用下面一行代码即可

```
CharBuffer.wrap(source_code);
```

于是我们自定义的 JavaFileObject 如下

```java
SimpleJavaFileObject sourceJavaFileObject = new SimpleJavaFileObject(URI.create(className + ".java"),
                JavaFileObject.Kind.SOURCE){
  public CharBuffer getCharContent(boolean b) {
    return CharBuffer.wrap(source_code);
  }
};
```

如果有多个要编译的类，可以写一个工具返回 JavaFileObject 数组作为 compilationUnits 然后传给 getTask()。

总结一下，其实 Java 编译器的工作原理简单说就是输入源码，输出字节码的过程，而源码与字节码的对象都是 JavaFileObject 实例。当我们需要修改源码的输入方式或者字节码的输出方式的时候，就需要建立自己的 JavaFileObject 子类来定义特定的行为。