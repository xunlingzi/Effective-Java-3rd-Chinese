## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 9: Prefer try-with-resources to try-finally（使用 try-with-resources 优于 try-finally）

The Java libraries include many resources that must be closed manually by invoking a close method. Examples include InputStream,OutputStream, and java.sql.Connection. Closing resources is often overlooked by clients, with predictably dire performance consequences. While many of these resources use finalizers as a safety net, finalizers don’t work very well (Item 8).

Java 库包含许多必须通过调用 close 方法手动关闭的资源。比如 InputStream、OutputStream 和 java.sql.Connection。关闭资源常常会被客户端忽略，这会导致可怕的性能后果。虽然这些资源中的许多都使用finalizer 机制作为安全网，但 finalizer 并不能很好地工作（[Item-8](/Chapter-2/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md)）。*The Java libraries include many resources that must be closed manually by invoking a close method. Examples include InputStream, OutputStream, and java.sql.Connection. Closing resources if often overlooked by client, with predictably dire performance consequences. While many of these resources use finalizers as a safety net, finalizers don't work very well.*

Historically, a try-finally statement was the best way to guarantee that a resource would be closed properly, even in the face of an exception or return:

从以往来看，try-finally 语句是确保正确关闭资源的最佳方法，即使在程序抛出异常或返回时的情况下：*Historically, a try-finally statement was the best way to guarantee that a resource would be closed properly, even in the face of an exception or return:*

```java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

This may not look bad, but it gets worse when you add a second resource:

这可能看起来不错，但当你添加第二个资源时，情况会变得更糟：*This may not look bad, but it gets worse when you add a second resource:*

```java
// try-finally is ugly when used with more than one resource!
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
    try {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    } finally {
        out.close();
        }
    }
    finally {
        in.close();
    }
}
```

It may be hard to believe, but even good programmers got this wrong most of the time. For starters, I got it wrong on page 88 of Java Puzzlers [Bloch05], and no one noticed for years. In fact, two-thirds of the uses of the close method in the Java libraries were wrong in 2007.

这可能很难相信，但即使是优秀的程序员在大多数时候也会犯这个错误。首先，我在 Java Puzzlers [Bloch05]的 88 页上弄错了，多年来没有人注意到。事实上，2007 年 Java 类库中三分之二的 close 方法使用都是错误的。*It may be hard to believe, but even good programmers got this wrong most of the time. For starters, I got it wrong on page 88 of Java Puzzlers, and no one noticed for years. In fact, two-thirds of the uses of the close method in the Java libraries are wrong in 2007.*

**译注：《Java Puzzlers》的中文译本为《Java 解惑》**

Even the correct code for closing resources with try-finally statements,as illustrated in the previous two code examples, has a subtle deficiency. The code in both the try block and the finally block is capable of throwing exceptions. For example, in the firstLineOfFile method, the call to readLine could throw an exception due to a failure in the underlying physical device, and the call to close could then fail for the same reason. Under these circumstances, the second exception completely obliterates the first one. There is no record of the first exception in the exception stack trace, which can greatly complicate debugging in real systems—usually it’s the first exception that you want to see in order to diagnose the problem. While it is possible to write code to suppress the second exception in favor of the first, virtually no one did because it’s just too verbose.

使用 try-finally 语句关闭资源的正确代码（如前两个代码示例所示）也有一个细微的缺陷。try 和 finally 语句块中的代码都能够抛出异常。*Even the correct code for closing resource with try-finally statements, as illustrated in the previous two code examples, has a subtle deficiency. The code in both the try block and the finally block is capable of throwing exceptions.*

例如，在 firstLineOfFile 方法中，由于底层物理设备发生故障，对 readLine 的调用可能会抛出异常，而 close 方法的调用也可能出于同样的原因而失败。*For example, in the firstLineOfFile method, the call to readLine could throw an exception due to failure in the underlying physical device, and the call to close could then fail for the same reason.*

在这种情况下，第二个异常将完全覆盖第一个异常。在异常堆栈跟踪中没有第一个异常的记，这可能会使实际系统中的调试变得非常复杂——通常你可能是希望看到第一个异常，以便诊断问题。*Under these circumstances, the second exception completely **obliterates<抹去，消除>** the first one. There is no record of the first exception in the exception stack trace, which can greatly complicate debugging in real systems - usually it's the first exception that you want to see in order to diagnose the problem.*

虽然可以通过编写代码来抑制第二个异常从而支持第一个异常，但实际上没有人这样做，因为它太过冗长。*While it is possible to write code to suppress the second exception **in favor of<支持，有利于>** the first, virtually no one did because it's just too verbose.*

All of these problems were solved in one fell swoop when Java 7 introduced the try-with-resources statement [JLS, 14.20.3]. To be usable with this construct, a resource must implement the AutoCloseable interface, which consists of a single void-returning close method. Many classes and interfaces in the Java libraries and in third-party libraries now implement or extend AutoCloseable. If you write a class that represents a resource that must be closed, your class should implement AutoCloseable too.

当 Java 7 引入 try-with-resources 语句 [JLS, 14.20.3]时，所有这些问题都一下子得到了解决。要使用这个结构，资源必须实现 AutoCloseable 接口，它仅由一个返回void的 close 方法组成。*All of these problems were solved **in one fell swoop<一举，一下子>** when Java 7 introduced the try-with-resources statement. To be usable with this construct, a resource must implement the AutoCloseable interface, which consists of a single void-returning close method.*

Java 类库和第三方库中的许多类和接口现在都实现或继承了 AutoCloseable 接口。如果你编写的类表示一个必须关闭的资源，那么也应该实现 AutoCloseable 接口。*Many classes and interfaces in the Java libraries and in third-party libraries now implement or extend AutoCloseable. If you write a class that represents a resource that must be closed, your class should implement AutoCloseable too.*

Here’s how our first example looks using try-with-resources:

下面是使用 try-with-resources 的第一个示例：*Here's how our first example looks using try-with-resources:*

```java
// try-with-resources - the the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```

And here’s how our second example looks using try-with-resources:

这是我们第二个示例使用 try-with-resources 的样子：*And here's how our second example look using try-with-resources:*

```java
// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
            out.write(buf, 0, n);
    }
}
```

Not only are the try-with-resources versions shorter and more readable than the originals, but they provide far better diagnostics. Consider the firstLineOfFile method. If exceptions are thrown by both the readLine call and the (invisible) close, the latter exception is suppressed in favor of the former. In fact, multiple exceptions may be suppressed in order to preserve the exception that you actually want to see. These suppressed exceptions are not merely discarded; they are printed in the stack trace with a notation saying that they were suppressed. You can also access them programmatically with the getSuppressed method, which was added to Throwable in Java 7.

和使用 try-finally 的原版代码相比，try-with-resources 不仅更简洁、更具可读性，而且为开发者提供了更好的诊断方式。考虑 firstLineOfFile 方法。如果 readLine 调用和（不可见的） close 同时抛出异常，则后一个异常将被抑制，以支持前一个异常。*Not only are the try-with-resources versions shorter and more readable than the originals, but they provide far better diagnostics. Consider the firstLineOfFile method. If exceptions are thrown by both the readLine call and the (invisible) close, the latter exception is suppressed in favor of the former.*

实际上，为了保留你真正想看到的异常，更多的异常可能会被抑制。这些被抑制的异常不会仅仅被抛弃；它们会被打印在堆栈跟踪中，并标记它们被抑制。可以通过编程方式使用 getSuppressed 方法访问到它们，该方法是在 Java 7 中添加到 Throwable 中的。*In fact, multiple exceptions may be suppressed in order to preserve the exception that you actually want to see. These suppressed exceptions are not merely discarded; they are printed in the stack trace with a notation saying that they were suppressed. You can also access them programmatically with the getSuppressed method, which was added to Throwable in Java 7.*

You can put catch clauses on try-with-resources statements, just as you can on regular try-finally statements. This allows you to handle exceptions without sullying your code with another layer of nesting. As a slightly contrived example, here’s a version our firstLineOfFile method that does not throw exceptions, but takes a default value to return if it can’t open the file or read from it:

可以在 try-with-resources 语句中添加 catch 子句，就像在常规的 try-finally 语句一样。这允许处理异常时不必用另一层嵌套来污染代码。*You can put catch clauses on try-with-resources statements, just as you can on regular try-finally statements. This allows you to handle exceptions without **sullying<玷污>** your code with another layer of nesting.*

作为一个稍微有些做作的例子，下面是我们的 firstLineOfFile 方法的一个版本，它不抛出异常，但如果无法打开或读取文件，则返回一个默认值：*As a slightly **contrived<人为的，做作的>** example, here's a version our firstLineOfFile method that does not throw exceptions, but takes a default value to return if it can't open the file or read from it:*

```java
// try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

The lesson is clear: Always use try-with-resources in preference to try-finally when working with resources that must be closed. The resulting code is shorter and clearer, and the exceptions that it generates are more useful. The try-with-resources statement makes it easy to write correct code using resources that must be closed, which was practically impossible using try-finally.

结论很清楚：在使用必须关闭的资源时，总是优先使用 try-with-resources，而不是 try-finally。前者的代码更短、更清晰，生成的异常更有用。使用 try-with-resources 语句可以很容易地为必须关闭的资源编写正确的代码，而使用 try-finally 几乎是不可能的。*The lesson is clear: Always use try-with-resources in preference to try-finally when working with resources that must be closed. The resulting code is shorter and clearer, and the exceptions that it generates are more useful. The try-with-resources statement make it easy to write correct using resources that must be closed, which was practically impossible using try-finally.*

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-2/Chapter-2-Introduction.md)**
- **Previous Item（上一条目）：[Item 8: Avoid finalizers and cleaners（避免使用终结器和清除器）](/Chapter-2/Chapter-2-Item-8-Avoid-finalizers-and-cleaners.md)**
- **Next Item（下一条目）：[Chapter 3 Introduction（章节介绍）](/Chapter-3/Chapter-3-Introduction.md)**
