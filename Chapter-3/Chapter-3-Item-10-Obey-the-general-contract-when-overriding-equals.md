## Chapter 3. Methods Common to All Objects（对象的通用方法）

### Item 10: Obey the general contract when overriding equals（覆盖 equals 方法时应遵守的约定）

Overriding the equals method seems simple, but there are many ways to get it wrong, and consequences can be dire. The easiest way to avoid problems is not to override the equals method, in which case each instance of the class is equal only to itself. This is the right thing to do if any of the following conditions apply:

覆盖 equals 方法似乎很简单，但是有很多覆盖的方式会导致出错，而且后果可能非常严重。避免问题的最简单方法是不覆盖 equals 方法，在这种情况下，类的每个实例都只等于它自己。如果符合下列任何条件，就是正确的做法：*Overriding the equals method seems simple, but there are many ways to get it wrong, and consequences can be dire. The easiest way to avoid problems is not to override the equals method, in which case each instance of the class is equal only to itself. This is the right thing to do if any the following conditions apply:*

- **Each instance of the class is inherently unique.** This is true for classes such as Thread that represent active entities rather than values. The equals implementation provided by Object has exactly the right behavior for these classes.

**类的每个实例本质上都是唯一的。** 对于像 Thread 这样表示活动实体类而不是值类来说这是正确的。Object 提供的 equals 实现对于这些类具有完全正确的行为。*Each instance of the class is inherently unique. This is true for classes such as Thread that represent active entities rather than values. The equals implementation provided by Object has exactly the right behavior for these class.*

- **There is no need for the class to provide a “logical equality” test.** For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the equals implementation inherited from Object is ideal.

**类不需要提供一个「逻辑相等」的测试功能。** 例如，`java.util.regex.Pattern` 可以重写 equals 来检查两个 Pattern 实例是否表示完全相同的正则表达式，但设计人员认为客户不需要或不想要这个功能。在这种情况下，从 Object 继承的 equals 实现是理想的。*There is no need for the class to provide a "logical equality" test. For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn't think that clients would need or want this functionality. Under these circumstances, the equals implementation inherited from Object is ideal.*

- **A superclass has already overridden equals, and the superclass behavior is appropriate for this class.** For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations from AbstractList, and Map implementations from AbstractMap.

**父类已经重写了 equals 方法，则父类行为完全适用于子类。** 例如，大多数 Set 从 AbstractSet 继承了 equals 实现，List 从 AbstractList 继承实现，Map 从 AbstractMap 继承实现。*A superclass has already overridden equals, and the superclass behavior is appropriate for this class. For example, most Set implementations inherit their equals implementation from AbstractSet, List implementations form AbstractList, and Map implementations from AbstractMap.*

- **The class is private or package-private, and you are certain that its equals method will never be invoked.** If you are extremely risk-averse,you can override the equals method to ensure that it isn’t invoked accidentally:

**类是私有的或包级私有的，并且你确信它的 equals 方法永远不会被调用。** 如果你非常厌恶风险，你可以覆盖 equals 方法，以确保它不会意外调用：*The class is private or package-private, and you are certain that its equals method will never be invoked. If you are extremely risk-adverse, you can override the equals method to ensure that it isn't invoked accidentally:

```java
@Override
public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
}
```

So when is it appropriate to override equals? It is when a class has a notion of logical equality that differs from mere object identity and a superclass has not already overridden equals. This is generally the case for value classes. A value class is simply a class that represents a value, such as Integer or String. A programmer who compares references to value objects using the equals method expects to find out whether they are logically equivalent, not whether they refer to the same object. Not only is overriding the equals method necessary to satisfy programmer expectations, it enables instances to serve as map keys or set elements with predictable, desirable behavior.

那么什么时候适合重写 equals 方法呢？当一个类有一个逻辑相等的概念，而这个概念不同于单纯的对象标识，且它的父类还没有重写 equals 方法时。*So when is it appropriate to override equals? It is when a class has a notion of logical equality that differs from mere object identity and a superclass has not already overridden equals.*

这通常用在值类的情况。值类是一个只表示值的类，例如 Integer 或 String。程序员使用 equals 方法比较值对象的引用，是希望比较它们逻辑上是否相等，而不是它们是否引用相同的对象。*This is generally the case for value classes. A value class is simply a class that represents a value, such as Integer or String. A programmer who compares references to value objects using the equals method expects to find out whether they are logically equivalent, not whether they refer to the same object.*

重写 equals 方法不仅是为了满足程序员的期望所必需的，它还使实例能够作为 Map 的键或作为 Set 元素时，具有可预测的、理想的行为。*Not only is overriding the equals method necessary to satisfy programmer expectations, it enables instances to serve as map keys or set elements with predictable, desirable behavior.*

**译注 1：有一个表示状态的内部类。没有覆盖 equals 方法时，equals 的结果与 s1==s2 相同，为 false，即两者并不是相同对象的引用。**
```java
public static void main(String[] args) {

    class Status {
        public String status;
    }

    Status s1 = new Status();
    Status s2 = new Status();

    System.out.println(s1==s2); // false
    System.out.println(s1.equals(s2)); // false
}
```
**译注 2：覆盖 equals 方法后，以业务逻辑来判断是否相同，具备相同 status 字段即为相同。在使用去重功能时，也以此作为判断依据。**
```java
public static void main(String[] args) {

    class Status {
        public String status;

        @Override
        public boolean equals(Object o) {
            return Objects.equals(status, ((Status) o).status);
        }
    }

    Status s1 = new Status();
    Status s2 = new Status();

    System.out.println(s1==s2); // false
    System.out.println(s1.equals(s2)); // true
}
```

One kind of value class that does not require the equals method to be overridden is a class that uses instance control (Item 1) to ensure that at most one object exists with each value. Enum types (Item 34) fall into this category. For these classes, logical equality is the same as object identity, so Object’s equals method functions as a logical equals method.

有一种不需要重写 equals 方法的类是使用实例控制（[Item-1](/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）来确保每个值最多只存在一个对象的类。枚举类型（[Item-34](/Chapter-6/Chapter-6-Item-34-Use-enums-instead-of-int-constants.md)）属于这一类别。对于这些类，逻辑相等与对象标识相同，因此对象的 equals 方法与逻辑 equals 方法相同。*One kind of value class that does not require the equals method to be overridden is a class that uses instance control (Item 1) to ensure that at most one object exists with each value. Enum types (Item 34) fall into this category. For these classes, logical equality is the same as object identity, so Object's equals method functions as a logical equals method.*

When you override the equals method, you must adhere to its general contract. Here is the contract, from the specification for Object :

当你覆盖 equals 方法时，你必须遵守它的通用约定。以下是具体内容，来自 Object 规范：*When you override the equals method, you must adhere to its general contract. Here is the contract, from the specification for Object:*

The equals method implements an equivalence relation. It has these properties:

equals 方法实现了等价关系。它应有这些属性：*The equals method implements an equivalence relation. It has these properties:*

- Reflexive: For any non-null reference value x, x.equals(x) must return true.

反身性：对于任何非空引用 x，`x.equals(x)` 必须返回 true。*Reflexive: For any non-null reference value x, x.equals(x) must return true.*

- Symmetric: For any non-null reference values x and y, x.equals(y) must return true if and only if y.equals(x) returns true.

对称性：对于任何非空引用 x 和 y，`x.equals(y)` 当且仅当 `y.equals(x)` 返回 true 时返回 true。*Symmetric: For any non-null reference values x and y, x.equals(y) must return true if and only if y.equals(x) returns true.*

- Transitive: For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(z) returns true, then x.equals(z) must return true.

传递性：对于任何非空引用 x, y, z，如果 `x.equals(y)` 返回 true，`y.equals(z)` 返回 true，那么 `x.equals(z)` 必须返回 true。*Transitive: For any non-null reference values x, y, z, if x.equals(y) returns true and y.equals(x) returns true, then x.equals(z) must return true.*

- Consistent: For any non-null reference values x and y, multiple invocations of x.equals(y) must consistently return true or consistently return false, provided no information used in equals comparisons is modified.

一致性：对于任何非空引用 x 和 y,  如果在 equals 比较中使用的信息没有修改，则多次调用 `x.equals(y)` 必须始终返回 true 或始终返回 false。*Consistent: For any non-null reference values x and y, multiple invocations of x.equals(y) must consistently return true or consistently return false, provided no information used in equals comparisons is modified.*

- For any non-null reference value x, x.equals(null) must return false.

对于任何非空引用 x，`x.equals(null)` 必须返回 false。*For any non-null reference value x, x.equals(null) must return false.*

Unless you are mathematically inclined, this might look a bit scary, but do not ignore it! If you violate it, you may well find that your program behaves erratically or crashes, and it can be very difficult to pin down the source of the failure. To paraphrase John Donne, no class is an island. Instances of one class are frequently passed to another. Many classes, including all collections classes,depend on the objects passed to them obeying the equals contract.

除非你喜欢数学，否则这看起来有点可怕，但不要忽略它！如果你违反了它，你的程序很可能会出现行为异常或崩溃，并且很难确定故障的根源。*Unless you are mathematically inclined, this might look a bit scary, but do not ignore it! If you violate it, you may well find that your program behaves **erratically<不规律的，不稳定的>** or crashes, and it can be very difficult to **pin down<确定，固定>** the source of the failure.*

用 John Donne 的话来说，没有一个类是孤立的。一个类的实例经常被传递给另一个类。许多类。包括所有集合类，都依赖于传递给它们的遵守 equals 约定的对象。*To paraphrase John Donne, no class is an island. Instances of one class are frequently passed to another. Many classes, including all collections classes, depend on the objects passed to them obeying the equals contract.*

Now that you are aware of the dangers of violating the equals contract, let’s go over the contract in detail. The good news is that, appearances notwithstanding, it really isn’t very complicated. Once you understand it, it’s not hard to adhere to it.

既然你已经意识到了违反 equals 约定的危险，让我们详细讨论一下该约定。好消息是，尽管表面上看起来很复杂，但其实并非如此。一旦你理解了，就不难遵守这一约定。*Now that you are aware of the dangers of violating the equals contract, let's go over the contract in detail. The good news is that, appearances notwithstanding, it really isn't very complicated. Once you understand it, it's not hard to adhere to it.*

So what is an equivalence relation? Loosely speaking, it’s an operator that partitions a set of elements into subsets whose elements are deemed equal to one another. These subsets are known as equivalence classes. For an equals method to be useful, all of the elements in each equivalence class must be interchangeable from the perspective of the user. Now let’s examine the five requirements in turn:

什么是等价关系？简单地说，它是一个操作符，它将一组元素划分为子集，子集的元素被认为是彼此相等的。这些子集被称为等价类。*So what is an equivalence relation? Loosely speaking, it's an operator that partitions a set of elements into subsets whose elements are deemed equal to one another. These subsets are known as equivalence classes.*

为了使 equals 方法有用，从用户的角度来看，每个等价类中的所有元素都必须是可互换的。现在让我们依次检查以下五个要求：*For an equals method to be useful, all of the elements in each equivalence class must be **interchangeable<可互换的>** from the perspective of the user. Not let's examine the five requirements in turn:*

**Reflexivity** —The first requirement says merely that an object must be equal to itself. It’s hard to imagine violating this one unintentionally. If you were to violate it and then add an instance of your class to a collection, the contains method might well say that the collection didn’t contain the instance that you just added.

**反身性** ，第一个要求只是说一个对象必须与自身相等。很难想象会无意中违反了这条规则。如果你违反了它，然后将类的一个实例添加到集合中，contains 方法很可能会说该集合不包含你刚才添加的实例。*Reflexivity - The first requirement says merely that an object must be equal to itself. It's hard to imagine violating this one unintentionally. If you were to violate it and then an instance of you class to a collection, the contains method might well say that the collection didn't contain the instance that you just added.*

**Symmetry** —The second requirement says that any two objects must agree on whether they are equal. Unlike the first requirement, it’s not hard to imagine violating this one unintentionally. For example, consider the following class,which implements a case-insensitive string. The case of the string is preserved by toString but ignored in equals comparisons:

**对称性** ，第二个要求是任何两个对象必须在是否相等的问题上达成一致。与第一个要求不同，无意中违反了这个要求的情况不难想象。例如，考虑下面的类，它实现了不区分大小写的字符串。字符串的大小写由 toString 保留，但在 equals 比较中被忽略：*Symmetry - The second requirement says that any two objects must agree on whether they are equal. Unlike the first requirement, it's not hard to imagine violating this one unintentionally. For example, consider the following class, which implements a case-insensitive string. The case of the string is preserved by toString but ignored in equals comparisons.*

```java
// Broken - violates symmetry!
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
	}

    // Broken - violates symmetry!
    @Override
    public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
        return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);

        if (o instanceof String) // One-way interoperability!
            return s.equalsIgnoreCase((String) o);

        return false;
    } ... // Remainder omitted
}
```

The well-intentioned equals method in this class naively attempts to interoperate with ordinary strings. Let’s suppose that we have one caseinsensitive string and one ordinary one:

这个类中的 equals 方法天真地尝试与普通字符串进行互操作。假设我们有一个不区分大小写的字符串和一个普通字符串：*The well-intentioned equals method in this class naively attempts to interoperate with ordinary strings. Let's suppose that we have one caseinsensitive string and one ordinary one:

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String s = "polish";
```

As expected, cis.equals(s) returns true. The problem is that while the equals method in CaseInsensitiveString knows about ordinary strings, the equals method in String is oblivious to case-insensitive strings.Therefore, s.equals(cis) returns false, a clear violation of symmetry. Suppose you put a case-insensitive string into a collection:

正如预期的那样，`cis.equals(s)` 返回 true。问题是，虽然 CaseInsensitiveString 中的 equals 方法知道普通字符串，但是 String 中的 equals 方法却不了解不区分大小写的字符串。因此，`s.equals(cis)` 返回 false，这明显违反了对称性。假设你将一个不区分大小写的字符串放入集合中：*As expected, cis.equals(s) returns true. The problem is that while the equals method in the equals method in CaseinsensitiveString knows about ordinary strings, the equals method in String is **oblivious<健忘的；不注意的；不知道的>** to case-insensitive strings. Therefore, s.equals(cis) returns false, a clear violation of symmetry. Suppose you put a case-insensitive string into a collection:*

```java
List<CaseInsensitiveString> list = new ArrayList<>();
list.add(cis);
```

What does list.contains(s) return at this point? Who knows? In the current OpenJDK implementation, it happens to return false, but that’s just an implementation artifact. In another implementation, it could just as easily return true or throw a runtime exception. **Once you’ve violated the equals contract, you simply don’t know how other objects will behave when confronted with your object.**

此时 `list.contains(s)` 返回什么？谁知道呢？在当前的 OpenJDK 实现中，它碰巧返回 false，但这只是一个实现案例。在另一个实现中，它可以很容易地返回 true 或抛出运行时异常。一旦你违反了 equals 约定，就不知道当其他对象面对你的对象时会如何表现了。*What does list.contains(s) return at this point? Who knows? In the current OpenJDK implementation, it happens to return false, but that's just an implementation artifact. In another implementation, it could just as easily return true or throw a runtime exception. Once you've violated the equals contract, you simply don't know how other objects will behave when confronted with your object.*

**译注：contains 方法在 ArrayList 中的实现源码如下（省略了源码中的多行注释）：**
```java
// ArrayList 的大小
private int size;

// 保存 ArrayList 元素的容器，一个 Object 数组
transient Object[] elementData; // non-private to simplify nested class access

public boolean contains(Object o) {
    return indexOf(o) >= 0;
}

public int indexOf(Object o) {
    return indexOfRange(o, 0, size);
}

int indexOfRange(Object o, int start, int end) {
    Object[] es = elementData;
    if (o == null) {
        for (int i = start; i < end; i++) {
            if (es[i] == null) {
                return i;
            }
        }
    } else {
        for (int i = start; i < end; i++) {
            if (o.equals(es[i])) {
                return i;
            }
        }
    }
    return -1;
}
```

To eliminate the problem, merely remove the ill-conceived attempt to interoperate with String from the equals method. Once you do this, you can refactor the method into a single return statement:

为了消除这个问题，只需从 equals 方法中删除与 String 互操作的错误尝试。这样做之后，你可以重构方法为一个单一的返回语句：*To eliminate the problem, merely remove the ill-conceived attempt to interoperate with String from the equals method. Once you do this, you can refactor the method into a single return statement:*

```java
@Override
public boolean equals(Object o) {
    return o instanceof CaseInsensitiveString && ((CaseInsensitiveString) o).s.equalsIgnoreCase(s);
}
```

**Transitivity** —The third requirement of the equals contract says that if one object is equal to a second and the second object is equal to a third, then the first object must be equal to the third. Again, it’s not hard to imagine violating this requirement unintentionally. Consider the case of a subclass that adds a new value component to its superclass. In other words, the subclass adds a piece of information that affects equals comparisons. Let’s start with a simple immutable two-dimensional integer point class:

**传递性** ，equals 约定的第三个要求是，如果一个对象等于第二个对象，而第二个对象等于第三个对象，那么第一个对象必须等于第三个对象。同样，无意中违反了这个要求的情况不难想象。*Transitivity - The third requirement of the equals contract says that if one object is equal to a second and the second object is equal to a third, then the first object must be equal to the third. Again, it's not hard to imagine violating this requirement unintentionally.*

考虑向父类添加新的值组件时，子类的情况。换句话说，子类添加了一条影响 equals 比较的信息。让我们从一个简单的不可变二维整数 Point 类开始：*Consider the case of a subclass that adds a new value component to its superclass. In other words, the subclass adds a piece of information that affects equals comparisons. Let's start with a simple immutable two-dimensional integer point class:*

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point)o;
        return p.x == x && p.y == y;
    }
    ... // Remainder omitted
}
```

Suppose you want to extend this class, adding the notion of color to a point:

假设你想继承这个类，将颜色的概念添加到点类中：*Suppose you want to extend this class, adding the notion of color to a point:*

```java
public class ColorPoint extends Point {
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
    ... // Remainder omitted
}
```

How should the equals method look? If you leave it out entirely, the implementation is inherited from Point and color information is ignored in equals comparisons. While this does not violate the equals contract, it is clearly unacceptable. Suppose you write an equals method that returns true only if its argument is another color point with the same position and color:

equals 方法应该是什么样子？如果你完全忽略它，则实现将从 Point 类继承而来，在 equals 比较中颜色信息将被忽略。虽然这并不违反 equals 约定，但显然是不可接受的。假设你写了一个 equals 方法，该方法只有当它的参数是另一个 ColorPoint，且位置和颜色都相同时才返回 true：*How should the equals method look? If you leave it out entirely, the implementation is inherited from Point and color information is ignored in equals comparisons. While this does not violate the equals contract, it is clearly unacceptable. Suppose you write an equals method that returns true only if its argument is another color point with the same position and color:*

```java
// Broken - violates symmetry!
@Override
public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

The problem with this method is that you might get different results when comparing a point to a color point and vice versa. The former comparison ignores color, while the latter comparison always returns false because the type of the argument is incorrect. To make this concrete, let’s create one point and one color point:

这个方法的问题是，当你比较一个 Point 对象和一个 ColorPoint 对象时，你可能会得到不同的结果，反之亦然。前者比较忽略颜色，而后者比较总是返回 false，因为参数的类型是不正确的。为了使问题更具体，让我们创建一个 Point 对象和一个 ColorPoint 对象：*The problem with this method is that you might get different results when comparing a point to a color point and **vice versa<反之亦然>**. The former comparison ignores color, while the latter comparison always returns false because the type of the argument is incorrect. To make this concrete, let's create one point and one color point:*

```java
Point p = new Point(1, 2);
ColorPoint cp = new ColorPoint(1, 2, Color.RED);
```

Then p.equals(cp) returns true, while cp.equals(p) returns false. You might try to fix the problem by having ColorPoint.equals ignore color when doing “mixed comparisons”:

然后，`p.equals(cp)` 返回 true，而 `cp.equals(p)` 返回 false。您可以尝试通过让 ColorPoint.equals 在进行“混合比较”时忽略颜色来解决该问题：*Then p.equals(cp) returns true, while cp.equals(p) returns false. You might try to fix the problem by having ColorPoint.equals ignore color when doing "mixed comparisons":*

```
// Broken - violates transitivity!
@Override
public boolean equals(Object o) {
    if (!(o instanceof Point))
        return false;

    // If o is a normal Point, do a color-blind comparison
    if (!(o instanceof ColorPoint))
        return o.equals(this);

    // o is a ColorPoint; do a full comparison
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

This approach does provide symmetry, but at the expense of transitivity:

这种方法确实提供了对称性，但牺牲了传递性：*This approach does provide symmetry, but **at the expense of<以…为代价>** transitivity:*

```
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

Now p1.equals(p2) and p2.equals(p3) return true, while p1.equals(p3) returns false, a clear violation of transitivity. The first two comparisons are “color-blind,” while the third takes color into account.

现在，`p1.equals(p2)` 和 `p2.equals(p3)` 返回 true，而 `p1.equals(p3)` 返回 false，这明显违反了传递性。前两个比较都不考虑颜色信息，而第三个比较考虑了颜色。*Now p1.equals(p2) and p2.equals(p3) return true, while p1.equals(p3) returns false, a clear violation of transitivity. The first two comparisons are "color-blind", while the third takes color into account.*

Also, this approach can cause infinite recursion: Suppose there are two subclasses of Point, say ColorPoint and SmellPoint, each with this sort of equals method. Then a call to myColorPoint.equals(mySmellPoint) will throw a StackOverflowError.

此外，这种方法会导致无限递归：假设有两个 Point 的子类，比如 ColorPoint 和 SmellPoint，每个都使用这种 equals 方法。然后调用 `myColorPoint.equals(mySmellPoint)` 会抛出 StackOverflowError 异常。*Also, this approach can cause infinite recursion: Suppose there are two subclasses of Point, say ColorPoint and SmellPoint, each with this sort of equals method. The a call to myColorPoint.equals(mySmellPoint) will throw a StackOverflowError.*

So what’s the solution? It turns out that this is a fundamental problem of equivalence relations in object-oriented languages. **There is no way to extend an instantiable class and add a value component while preserving the equals contract,** unless you’re willing to forgo the benefits of object-oriented abstraction.

那么解决方案是什么？ 事实证明，这是面向对象语言中等价关系的一个基本问题。**除非你愿意放弃面向对象抽象的好处，否则无法在保留equals约定的同时继承可实例化类并添加值组件。** *So what's the solution? It turns out that this is a fundamental problem of equivalence relations in object-oriented languages. There is no way to extend an instantiable class and add a value component while preserving the equals contract, unless you're willing to **forgo<放弃>** the benefits of object-oriented abstraction.*

You may hear it said that you can extend an instantiable class and add a value component while preserving the equals contract by using a getClass test in place of the instanceof test in the equals method:

你可能听说过，你可以扩展一个可实例化类并添加一个值组件，同时通过使用 getClass 测试代替 equals 方法中的 instanceof 测试来保留 equals 约定：*You may hear it said that you can extend an instantiable class and add a value component while preserving the equals contract by using a getClass test in place of the instanceof test in the equals method:*

```java
// Broken - violates Liskov substitution principle (page 43)
@Override
public boolean equals(Object o) {

    if (o == null || o.getClass() != getClass())
        return false;

    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

This has the effect of equating objects only if they have the same implementation class. This may not seem so bad, but the consequences are unacceptable: An instance of a subclass of Point is still a Point, and it still needs to function as one, but it fails to do so if you take this approach! Let’s suppose we want to write a method to tell whether a point is on the unit circle. Here is one way we could do it:

只有当对象具有相同的实现类时，才具有等效对象的效果。这可能看起来没那么糟糕，但后果是不可接受的：Point子类的实例仍然是Point，它仍然需要作为Point运行，但如果你采用这种方法，它就无法做到这一点！假设我们要写一个方法来判断一个点是否在单位圆上。我们可以这样做：*This has the effect of equating objects only if they have the same implementation class. This may not seem so bad, but the consequences are unacceptable: An instance of a subclass of Point is still a Point, and it still needs to function as one, but it fails to do so if you take this approach! Let's suppose we want to write a method to tell whether a point is on the unit circle. Here is one way we could do it:*

```java
// Initialize unitCircle to contain all Points on the unit circle
private static final Set<Point> unitCircle = Set.of(
        new Point( 1, 0), new Point( 0, 1),
        new Point(-1, 0), new Point( 0, -1)
    );

    public static boolean onUnitCircle(Point p) {
        return unitCircle.contains(p);
}
```

While this may not be the fastest way to implement the functionality, it works fine. Suppose you extend Point in some trivial way that doesn’t add a value component, say, by having its constructor keep track of how many instances have been created:

虽然这可能不是实现功能的最快方法，但它可以正常工作。假设你以一种不添加值组件的简单方式继承 Point，例如，让它的构造函数跟踪创建了多少实例：*While this may not be the faster way to implement the functionality, it works fine. Suppose you extend Point in some trivial way that doesn't add a value component, say, by having its constructor keep track of how many instances have been created:*

```java
public class CounterPoint extends Point {
    private static final AtomicInteger counter = new AtomicInteger();

    public CounterPoint(int x, int y) {
        super(x, y);
        counter.incrementAndGet();
    }

    public static int numberCreated() {
        return counter.get();
    }
}
```

The Liskov substitution principle says that any important property of a type should also hold for all its subtypes so that any method written for the type should work equally well on its subtypes [Liskov87]. This is the formal statement of our earlier claim that a subclass of Point (such as CounterPoint) is still a Point and must act as one. But suppose we pass a CounterPoint to the onUnitCircle method. If the Point class uses a getClass-based equals method, the onUnitCircle method will return false regardless of the CounterPoint instance’s x and y coordinates. This is so because most collections, including the HashSet used by the onUnitCircle method, use the equals method to test for containment, and no CounterPoint instance is equal to any Point. If, however, you use a proper instanceof-based equals method on Point, the same onUnitCircle method works fine when presented with a CounterPoint instance.

里氏替换原则指出，一个类型的任何重要属性都应该适用于所有子类型，因此为该类型编写的任何方法都应该在其子类型上同样有效 [Liskov87]。*The Liskov substitution principle says that any important property of a type should also hold for all its subtypes so that any method written for the type should work equally well on its subtypes.*

这是我们之前声明的一个正式陈述，即 Point 的子类（如 CounterPoint）仍然是一个 Point，必须作为一个 Point 类来看待。 但是，假设我们将一个 CounterPoint 对象传递给 onUnitCircle 方法。 如果 Point 类使用基于 getClass 的 equals 方法，则无论 CounterPoint 实例的 x 和 y 坐标如何，onUnitCircle 方法都将返回 false。 这*This is the formal statement of our earlier claim that a subclass of Point (such as CounterPoint) is still a Point and must act as one. But suppose we pass a CounterPoint to the onUnitCircle method. If the Point class uses a getClass-based equals method, the onUnitCircle method will return false regardless of the CounterPoint instance's x and y **coordinates<坐标>**.*

这是因为大多数集合（包括 onUnitCircle 方法使用的 HashSet）都使用 equals 方法来测试包含性，而CounterPoint 实例并不等于任何 Point 实例。 但是，如果您在 Point 上使用正确的基于 instanceof 的 equals 方法，则在与 CounterPoint 实例一起出现时，相同的 onUnitCircle 方法可以正常工作。*This is because most collections, including the HashSet used by the onUnitCircle method, use the equals method to test for containment, and not CounterPoint instance is equal to any Point. If, however, you use a proper instanceof-based equals method on Point, the same onUnitCircle method works fine when presented with a CounterPoint instance.

>  [!note]
>
> 里氏替换原则（Liskov Substitution Principle，LSP）面向对象设计的基本原则之一。里氏替换原则指出：任何父类可以出现的地方，子类一定可以出现。LSP 是继承复用的基石，只有当衍生类可以替换掉父类，软件单位的功能不受到影响时，父类才能真正被复用，而衍生类也能够在父类的基础上增加新的行为。

While there is no satisfactory way to extend an instantiable class and add a value component, there is a fine workaround: Follow the advice of Item 18,“Favor composition over inheritance.” Instead of having ColorPoint extend Point, give ColorPoint a private Point field and a public view method (Item 6) that returns the point at the same position as this color point:

虽然没有令人满意的方法来继承一个可实例化的类并添加一个值组件，但是有一个很好的变通方案：按照 [Item-18](/Chapter-4/Chapter-4-Item-18-Favor-composition-over-inheritance.md) 的建议，「优先使用组合而不是继承」。给 ColorPoint 一个私有的 Point 属性和一个公共的视图方法（[Item-6](/Chapter-2/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)），而不是让 ColorPoint 继承 Point，该方法返回具有相同位置的 ColorPoint 对象：*While there is no satisfactory way to extend an instantiable class and add a value component, there is a fine workaround: Follow the advice of Item 18, "Favor composition over inheritance." Instead of having ColorPoint extend Point, give ColorPoint a private field and a public view method that returns the point at the same position as the color point:*

```java
// Adds a value component without violating the equals contract
public class ColorPoint {
    private final Point point;
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        point = new Point(x, y);
        this.color = Objects.requireNonNull(color);
    }

    /**
    * Returns the point-view of this color point.
    */
    public Point asPoint() {
        return point;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint))
            return false;

        ColorPoint cp = (ColorPoint) o;
        return cp.point.equals(point) && cp.color.equals(color);
    }
    ... // Remainder omitted
}
```

There are some classes in the Java platform libraries that do extend an instantiable class and add a value component. For example,java.sql.Timestamp extends java.util.Date and adds a nanoseconds field. The equals implementation for Timestamp does violate symmetry and can cause erratic behavior if Timestamp and Date objects are used in the same collection or are otherwise intermixed. The Timestamp class has a disclaimer cautioning programmers against mixing dates and timestamps. While you won’t get into trouble as long as you keep them separate, there’s nothing to prevent you from mixing them, and the resulting errors can be hard to debug. This behavior of the Timestamp class was a mistake and should not be emulated.

Java 库中有一些类确实继承了一个可实例化的类并添加了一个值组件。例如，`java.sql.Timestamp` 继承 `java.util.Date` 并添加了 nanoseconds  字段。Timestamp 的 equals 实现确实违反了对称性，如果 Timestamp 和 Date 对象在同一集合中使用或以其他方式混合使用，则可能会导致不稳定的行为。*There are some classes in the Java platform libraries that do extend an instantiable class and add a value component. For example, java.sql.Timestamp extends java.util.Date and adds a nanoseconds field. The equals implementation for Timestamp does violate symmetry and can cause erratic behavior if Timestamp and Date objects are used in the same collection or are otherwise intermixed.*

Timestamp 类有一个免责声明，警告程序员不要混合使用日期和时间戳。虽然只要将它们分开，就不会遇到麻烦，但是没有什么可以阻止你将它们混合在一起，因此产生的错误可能很难调试。时间戳类的这种行为是错误的，不应该效仿。*The Timestamp class has a **disclaimer<免责声明>** cautioning programmers against mixing dates and timestamps. While you won't get into trouble as long as you keep them separate, there's nothing to prevent you from mixing them, and the resulting errors can be hard to debug. This behavior of the Timestamp class was a mistake and should not be emulated.*

Note that you can add a value component to a subclass of an abstract class without violating the equals contract. This is important for the sort of class hierarchies that you get by following the advice in Item 23, “Prefer class hierarchies to tagged classes.” For example, you could have an abstract class Shape with no value components, a subclass Circle that adds a radius field, and a subclass Rectangle that adds length and width fields.Problems of the sort shown earlier won’t occur so long as it is impossible to create a superclass instance directly.

注意，你可以将值组件添加到抽象类的子类中，而不会违反 equals 约定。 这对于你通过遵循第 23 条目中“优先使用类层次结构而不是标记类”的建议获得的类层次结构非常重要。*Note that you can add a value component to a subclass of an abstract class without violating the equals contract. This is important for the sort of class hierarchies that you get by following the advice in Item 23, "Prefer class hierarchies to tagged classes."*

例如，可以有一个没有值组件的抽象类 Shape，一个添加 radius 属性的子类 Circle 和一个添加 length 和 width 属性的子类 Rectangle。只要不能直接创建父类实例，前面所示的问题就不会发生。*For example, you could have an abstract class Shape with no value components, a subclass Circle that adds a radius field, and a subclass Rectangle that adds length and width fields. Problems of the sort shown earlier won't occur so long as it is impossible to create a superclass instance directly.*

**Consistency—** The fourth requirement of the equals contract says that if two objects are equal, they must remain equal for all time unless one (or both) of them is modified. In other words, mutable objects can be equal to different objects at different times while immutable objects can’t. When you write a class,think hard about whether it should be immutable (Item 17). If you conclude that it should, make sure that your equals method enforces the restriction that equal objects remain equal and unequal objects remain unequal for all time.

**一致性** — equals 条约的第四个要求是，如果两个对象相等，则它们必须始终保持相等，除非其中一个（或两个）被修改。 换句话说，可变对象可以在不同时间等于不同的对象，而不可变对象则不能。*Consistency - The fourth requirement of the equals contract says that if two objects are equal, the must remain equal for all time unless one (or both) of them is modified. In other words, mutable objects can be equal to different objects at different time while immutable objects can't.*

当你编写一个类时，请认真考虑它是否应该是不可变的。 如果你认为应该这样做，请确保你的 equals 方法强制执行以下限制：相等的对象始终保持相等，不相等的对象始终保持不相等。*When you write a class, think  hard about whether it should be immutable. If you conclude that it should, make sure that your equals method enforces the restriction that equal objects remain equal and unequal objects remain unequal all time.*

Whether or not a class is immutable, **do not write an equals method that depends on unreliable resources.** It’s extremely difficult to satisfy the consistency requirement if you violate this prohibition. For example, java.net.URL’s equals method relies on comparison of the IP addresses of the hosts associated with the URLs. Translating a host name to an IP address can require network access, and it isn’t guaranteed to yield the same results over time. This can cause the URL equals method to violate the equals contract and has caused problems in practice. The behavior of URL’s equals method was a big mistake and should not be emulated. Unfortunately, it cannot be changed due to compatibility requirements. To avoid this sort of problem,equals methods should perform only deterministic computations on memoryresident objects.

无论一个类是否不可变，都不要编写依赖于不可靠资源的 equals 方法。如果你违反了这个禁令，就很难满足一致性要求。*Whether or not a class is immutable, do not write an equals method that depends on unreliable resources. It's extremely difficult to satisfy the consistency requirement if you violate this prohibition.*

例如，`java.net.URL` 的 equals 方法依赖于与 url 相关联的主机的 IP 地址的比较。将主机名转换为 IP 地址可能需要网络访问，而且不能保证随着时间的推移产生相同的结果。这可能会导致 URL 的 equals 方法违反 equals 约定，并在实践中造成问题。*For example, java.net.URL's equals method relies on comparison of the IP addresses of the hosts associated with the URLs. Translating a host name to an IP address can require network access, and it isn't guaranteed to yield the same results over time. This can cause the URL equals method to violate the equals contract and has caused problems in practice.*

URL 的 equals 方法的行为是一个很大的错误，不应该被效仿。不幸的是，由于兼容性需求，它不能更改。为了避免这种问题，equals 方法应该只对内存驻留对象执行确定性计算。*The behavior of URL's equals method was a big mistake and should not be emulated. Unfortunately, it cannot be changed due to compatibility requirements. To avoid this sort of problem, equals methods should perform only **deterministic<确定性的>** computations on memoryresident objects.*

**Non-nullity—** The final requirement lacks an official name, so I have taken the liberty of calling it “non-nullity.” It says that all objects must be unequal to null. While it is hard to imagine accidentally returning true in response to the invocation o.equals(null), it isn’t hard to imagine accidentally throwing a NullPointerException. The general contract prohibits this. Many classes have equals methods that guard against it with an explicit test for null:

**非空性** ，最后的要求没有一个正式的名称，所以我冒昧地称之为「非空性」。意思是说所有对象都必须不等于 null。*Non-nullity - The final requirement lacks an official names, so I have **taken the liberty of<冒昧地>** calling it "non-nullity". It says that all objects must be unequal to null.*

虽然很难想象在响应调用 `o.equals(null)` 时意外地返回 true，但不难想象意外地抛出 NullPointerException 异常。一般约定中禁止这样做。许多类的 equals 方法都会通过显式的 null 测试来防止它：*While it is hard to imagine accidentally returning true in response to the invocation o.equals(null), it isn't hard to imagine accidentally throwing a NullPointerException. The general contract prohibits this. Many classes have equals methods that guard against it with an explicit test for null:*

```java
@Override
public boolean equals(Object o) {
    if (o == null)
        return false;
    ...
}
```

This test is unnecessary. To test its argument for equality, the equals method must first cast its argument to an appropriate type so its accessors can be invoked or its fields accessed. Before doing the cast, the method must use the instanceof operator to check that its argument is of the correct type:

这个判断是不必要的。要测试其参数是否相等，equals 方法必须首先将其参数转换为合适的类型，以便能够调用其访问器或访问其字段。在执行类型转换之前，方法必须使用 instanceof 运算符来检查其参数的类型是否正确：*This test is unnecessary. To test its argument for equality, the equals method must first csat its argument to an appropriate type so its accessors can be invoked or its fields accessed. Before doing the cast, the method must use the instanceof operator to check that its argument is of the correct type:*

```java
@Override
public boolean equals(Object o) {
    if (!(o instanceof MyType))
        return false;
    MyType mt = (MyType) o;
    ...
}
```

If this type check were missing and the equals method were passed an argument of the wrong type, the equals method would throw a ClassCastException, which violates the equals contract. But the instanceof operator is specified to return false if its first operand is null,regardless of what type appears in the second operand [JLS, 15.20.2]. Therefore,the type check will return false if null is passed in, so you don’t need an explicit null check.

如果缺少这个类型检查，并且 equals 方法传递了一个错误类型的参数，equals 方法将抛出 ClassCastException 异常，这违反了 equals 约定。但是，如果 instanceof 操作符的第一个操作数为 null，则指定该操作符返回 false，而不管第二个操作数 [JLS, 15.20.2] 中出现的是什么类型。因此，如果传入 null，类型检查将返回 false，因此不需要显式的 null 检查。*If this type check were missing and the equals method were passed an argument of the wrong type, the equals method would throw a ClassCastException, which violates the equals contract. But the instanceof operator is specified to return false if its first operand is null, regardless of what type appears in the second operand. Therefore, the type check will return false if null is passed in, so you don't need an explicit null check.*

Putting it all together, here’s a recipe for a high-quality equals method:

综上所述，以下是构建高质量 equals 方法的秘诀：*Putting it all together, here's a recipe for a high-quality equals method:*

1、**Use the == operator to check if the argument is a reference to this object.** If so, return true. This is just a performance optimization but one that is worth doing if the comparison is potentially expensive.

**使用 == 运算符检查参数是否是对该对象的引用。** 如果是，返回 true。这只是一种性能优化，但如果比较的代价可能很昂贵的话，那么值得这样做。*Use the == operator to check if the argument is a reference to this object. If so, return true. This is just a performance optimization but one that is worth doing if the comparison is potentially expensive.*

2、**Use the instanceof operator to check if the argument has the correct type.** If not, return false. Typically, the correct type is the class in which the method occurs. Occasionally, it is some interface implemented by this class. Use an interface if the class implements an interface that refines the equals contract to permit comparisons across classes that implement the interface. Collection interfaces such as Set, List, Map, and Map.Entry have this property.

**使用 instanceof 运算符检查参数是否具有正确的类型。** 如果不是，返回 false。通常，正确的类型是 equals 方法所在的那个类。有时候，它是由这个类实现的某个接口。如果类实现了一个接口，该接口对 equals 约定进行了改进，以允许实现该接口的类进行比较，则使用该接口。集合接口，如 Set、List、Map 和 Map.Entry 具有此特性。*Use the instanceof operator to check if the argument has the correct type. If not, return false. Typically, the correct type is the class in which the method occurs. Occasionally, it is some interface implemented by this class. Use an interface if the class implements an interface that refines the equals contract to permit comparisons across classes that implement the interface. Collection interfaces such as Set, List, Map, and Map.Entry have this property.*

3、**Cast the argument to the correct type.** Because this cast was preceded by an instanceof test, it is guaranteed to succeed.

**将参数转换为正确的类型。** 因为在这个类型转换之前有一个 instanceof 测试，所以它肯定会成功。*Cast the argument to the correct type. Because this cast was preceded by an instanceof test, it is guaranteed to succeed.*

4、**For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object.** If all these tests succeed, return true; otherwise, return false. If the type in Step 2 is an interface, you must access the argument’s fields via interface methods; if the type is a class, you may be able to access the fields directly, depending on their accessibility.

**对于类中的每个「重要」字段，检查参数的字段是否与该对象的相应字段匹配。** 如果所有这些测试都成功，返回 true；否则返回 false。如果第 2 步中的类型是接口，则必须通过接口方法访问参数的字段；如果类型是类，你也许可以直接访问字段，这取决于字段的访问权限。*For each "significant" field in the class, check if that field of the argument matches the corresponding field of this object. If all these tests succeed, return true; otherwise, return false. If the type in Step 2 is an interface, you must access the argument's fields via interface methods; if the type is a class, you may be able to access the fields directly, depending on their accessibility.*

For primitive fields whose type is not float or double, use the == operator for comparisons; for object reference fields, call the equals method recursively; for float fields, use the static Float.compare(float,float) method; and for double fields, use Double.compare(double, double). The special treatment of float and double fields is made necessary by the existence of Float.NaN, -0.0f and the analogous double values; see JLS 15.21.1 or the documentation of Float.equals for details. While you could compare float and double fields with the static methods Float.equals and Double.equals, this would entail autoboxing on every comparison, which would have poor performance. For array fields, apply these guidelines to each element. If every element in an array field is significant, use one of the Arrays.equals methods.

对于非 float 或 double 的基本类型，使用 == 运算符进行比较；对于对象引用字段，递归调用 equals 方法；对于 float 字段，使用静态 `Float.compare(float,float)` 方法；对于 double 字段，使用 `Double.compare(double, double)` 方法。*For primitive fields whose type is not float or double, use the == operator for comparisons; for object reference fields, call the equals method recursively; for float fields, use the static Float.compare(float, float) method; and for double fields, use Double.compare(double, double).*

由于 Float.NaN、-0.0f 和类似的 double 值的存在，需要对 float 和 double 字段进行特殊处理；请参阅 JLS 15.21.1 或 `Float.equals` 文档。虽然你可以使用静态方法 Float.equals 和 Double.equals 进行对 float 和 double 字段进行比较，但这需要每次比较时都进行自动装箱，会导致性能变差。*The special treatment of float and double fields is made necessary by the existence of Float.NaN, -0.0f and the analogous double values; see JLS 15.21.1 or the documentation of Float.equals for details. While you could compare float and double fields with the static methods Float.equals and Double.equals, this would entail autoboxing and every comparison, which would have poor performance.*

对于数组字段，将这些准则应用于每个元素。如果数组字段中的每个元素都很重要，那么使用重载的 `Arrays.equals` 方法之一。*For array fields, apply these guidelines to each element. If every element in an array field is significant, use one of the Array.equals methods.*

Some object reference fields may legitimately contain null. To avoid the possibility of a NullPointerException, check such fields for equality using the static method Objects.equals(Object, Object).

一些对象引用字段可能合法地包含 null。为了避免可能出现 NullPointerException，请使用静态方法 `Objects.equals(Object, Object)` 检查这些字段是否相等。*Some object reference fields may legitimately contain null. To avoid the possibility of a NullPointerException, check such fields for equality using the static method Objects.equals(Object, Object).*

For some classes, such as CaseInsensitiveString above, field comparisons are more complex than simple equality tests. If this is the case,you may want to store a canonical form of the field so the equals method can do a cheap exact comparison on canonical forms rather than a more costly nonstandard comparison. This technique is most appropriate for immutable classes (Item 17); if the object can change, you must keep the canonical form up to date.

对于某些类，例如上面的 CaseInsensitiveString，字段比较相对于简单的相等性测试要复杂得多。这这种情况下，你可能希望存储字段的规范形式，以便 equals 方法可以基于规范形式去做开销较小的精确比较，而不是开销很大的非标准比较。这种方式最适合于不可变类（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)）；如果对象可以更改，则必须使规范形式保持最新。*For some classes, such as CaseInsensitiveString above, field comparisons are more complex than simple equality tests. If this is the case, you may want to store a **canonical<规范的>** form of the field so the equals method can do a cheap exact comparison on canonical forms rather than a more costly nonstandard comparison. This technique is most appropriate for immutable classes; if the object can change, you must keep the canonical form up to date.*

The performance of the equals method may be affected by the order in which fields are compared. For best performance, you should first compare fields that are more likely to differ, less expensive to compare, or, ideally,both. You must not compare fields that are not part of an object’s logical state,such as lock fields used to synchronize operations. You need not compare derived fields, which can be calculated from “significant fields,” but doing so may improve the performance of the equals method. If a derived field amounts to a summary description of the entire object, comparing this field will save you the expense of comparing the actual data if the comparison fails.For example, suppose you have a Polygon class, and you cache the area. If two polygons have unequal areas, you needn’t bother comparing their edges and vertices.

equals 方法的性能可能会受到字段比较顺序的影响。 为了获得最佳性能，您应该首先比较更有可能不同的字段、比较成本较低的字段，或者理想情况下两者都比较。*The performance of the equals method may be affected by the order in which fields are compared. For best performance, you should first compare fields that are more likely to differ, less expensive to compare, or, ideally, both.*

您不得比较不属于对象逻辑状态的字段，例如用于同步操作的锁字段。您不需要比较派生字段，派生字段可以从“有效字段”中计算出来，但这样做可能会提高equals方法的性能。*You must not compare fields that are not part of an object's logical state, such as lock fields used to synchronize operations. You need not compare derived fields, which can be calculated form "significant fields", but doing so many improve the performance of the equals method..*

如果派生字段相当于整个对象的摘要描述，那么如果比较该字段失败，将节省比较实际数据的开销。例如，假设有一个多边形类，你缓存了它的面积。如果如果两个多边形的面积不相等，就不再需要比较它们的边和顶点。*If a derived field **amounts to<相当于>** a summary description of the entire object, comparing this field will save you the expense of comparing the actual data if  the comparison fails. For example, suppose you have a Polygon class, and you cache the area. If two polygons have unequal areas, you needn't bother comparing their edges and vertices.*

**When you are finished writing your equals method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent?** And don’t just ask yourself; write unit tests to check, unless you used AutoValue (page 49) to generate your equals method, in which case you can safely omit the tests. If the properties fail to hold, figure out why, and modify the equals method accordingly. Of course your equals method must also satisfy the other two properties (reflexivity and non-nullity), but these two usually take care of themselves.

**写完 equals 方法后，问自己三个问题：它具备对称性吗？具备传递性吗？具备一致性吗？** 不要只问自己，要编写单元测试来检查，除非使用 AutoValue（第 49 页）来生成 equals 方法，在这种情况下，你可以安全地省略测试。如果属性不成立，请找出原因，并相应地修改 equals 方法。当然，equals 方法还必须满足其他两个属性（反身性和非空性），但这两个属性通常会自动满足。*When you are finished writing your equals method, ask yourself three questions: Is it symmetric? Is it transitive? Is it consistent? And don't just ask yourself; write unit tests to check, unless you used AutoValue to generate your equals method, in which case you can safely omit the tests. If the properties fails to hold, figure out why, and modify the equals method accordingly. Of course your equals method must also satisfy the other two properties (reflexivity and non-nullity), but these two usually take care of themselves.*

An equals method constructed according to the previous recipe is shown in this simplistic PhoneNumber class:

在这个简单的 PhoneNumber 类中，根据前面的方法构造了一个 equals 方法：*An equals method constructed according to the previous recipe is shown in this simplistic PhoneNumber class:*

```java
// Class with a typical equals method
public final class PhoneNumber {
    private final short areaCode, prefix, lineNum;

    public PhoneNumber(int areaCode, int prefix, int lineNum) {
        this.areaCode = rangeCheck(areaCode, 999, "area code");
        this.prefix = rangeCheck(prefix, 999, "prefix");
        this.lineNum = rangeCheck(lineNum, 9999, "line num");
    }

    private static short rangeCheck(int val, int max, String arg) {
        if (val < 0 || val > max)
            throw new IllegalArgumentException(arg + ": " + val);
        return (short) val;
    }

    @Override
    public boolean equals(Object o) {
        if (o == this)
            return true;
        if (!(o instanceof PhoneNumber))
            return false;

        PhoneNumber pn = (PhoneNumber)o;
        return pn.lineNum == lineNum && pn.prefix == prefix && pn.areaCode == areaCode;
    } ... // Remainder omitted
}
```

Here are a few final caveats:

以下是一些最后的警告：

- **Always override hashCode when you override equals (Item 11).**

**当你覆盖 equals 时，也覆盖 hashCode。**（[Item-11](/Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)）*Always override hashCode when you override equals.*

- **Don’t try to be too clever.** If you simply test fields for equality, it’s not hard to adhere to the equals contract. If you are overly aggressive in searching for equivalence, it’s easy to get into trouble. It is generally a bad idea to take any form of aliasing into account. For example, the File class shouldn’t attempt to equate symbolic links referring to the same file. Thankfully, it doesn’t.

**不要自作聪明。** 如果你只是为了判断相等性而测试字段，那么遵循 equals 约定并不困难。如果你在寻求对等方面过于激进，则很容易陷入麻烦。一般来说，考虑到任何形式的别名都不是一个好主意。例如，File类不应试图将引用同一文件的符号链接等同起来。谢天谢地，它没有。*Don't try to be too clever. If you simply test fields for equality, it's not hard to adhere to the equals contract. If you are overly aggressive in searching for equivalence, it's easy to get into trouble. It is generally a bad idea to take any form of aliasing into account. For example, the File class shouldn't attempt to equate symbolic links referring to the same file. Thankfully, it doesn't.*

- **Don’t substitute another type for Object in the equals declaration.** It is not uncommon for a programmer to write an equals method that looks like this and then spend hours puzzling over why it doesn’t work properly:

**在 equals 的方法声明中，不要将参数 Object 替换成其他类型。** 对于程序员来说，编写一个类似于这样的 equals 方法，然后花上几个小时思考为什么它不能正常工作是很常见的：*Don't substitute another type for Object in the equals declaration. It is not uncommon for a programmer to write an equals method that looks like this and then spend hours puzzling over why it doesn't work properly:*

```java
// Broken - parameter type must be Object!
public boolean equals(MyClass o) {
    ...
}
```

The problem is that this method does not override Object.equals,whose argument is of type Object, but overloads it instead (Item 52). It is unacceptable to provide such a “strongly typed” equals method even in addition to the normal one, because it can cause Override annotations in subclasses to generate false positives and provide a false sense of security.

这里的问题是，这个方法没有重写其参数类型为 Object 的 Object.equals 方法，而是重载了它（[Item-52](/Chapter-8/Chapter-8-Item-52-Use-overloading-judiciously.md)）。即使是普通的方法，提供这样一个「强类型的」equals 方法是不可接受的，因为它会导致子类中的 Override 注解产生误报并提供错误的安全感。*The problem is that this method does not override Object.equals, whose argument is of type Object, but overloads it instead. It is unacceptable to provide such a "strongly typed" equals method even in addition to the normal one, because it can cause Override annotations in subclasses to generate false positives and provide a false sense of security.*

Consistent use of the Override annotation, as illustrated throughout this item, will prevent you from making this mistake (Item 40). This equals method won’t compile, and the error message will tell you exactly what is wrong:

如本条目所示，一致地使用 Override 注解将防止你犯此错误（[Item-40](/Chapter-6/Chapter-6-Item-40-Consistently-use-the-Override-annotation.md)）。这个 equals 方法不会编译，错误消息会告诉你到底出了什么问题：*Consistent use of the Override annotation, as illustrated throughout this item, will prevent you from making this mistake. This equals method won't compile, and the error message will tell you exactly what is wrong:*

```java
// Still broken, but won’t compile
@Override
public boolean equals(MyClass o) {
    ...
}
```

Writing and testing equals (and hashCode) methods is tedious, and the resulting code is mundane. An excellent alternative to writing and testing these methods manually is to use Google’s open source AutoValue framework, which automatically generates these methods for you, triggered by a single annotation on the class . In most cases, the methods generated by AutoValue are essentially identical to those you’d write yourself.

编写和测试 equals （和 hashCode）方法很乏味，生成的代码也很单调。手动编写和测试这些方法的一个很好的替代方法是使用谷歌的开源 AutoValue 框架，它会自动为你生成这些方法，只需在类上添加一个注解即可。在大多数情况下，AutoValue 生成的方法与你自己编写的方法本质上是相同的。*Writing and testing equals (and hashCode) methods is **tedious**, and the resulting code is **mundane<单调的，平凡的>**. An excellent alternative to writing and testing these methods manually is to use Google's open source AutoValue framework, which automatically generates these methods for you, triggered by a single annotation on the class. In most cases, the methods generated by AutoValue are essentially identical to those you'd write yourself.*

IDEs, too, have facilities to generate equals and hashCode methods, but the resulting source code is more verbose and less readable than code that uses AutoValue, does not track changes in the class automatically, and therefore requires testing. That said, having IDEs generate equals (and hashCode)methods is generally preferable to implementing them manually because IDEs do not make careless mistakes, and humans do.

IDE 也有生成 equals 和 hashCode 方法的功能，但是生成的源代码比使用 AutoValue 框架的代码更冗长，可读性更差，不会自动跟踪类中的变化，因此需要进行测试。也就是说，让 IDE 生成 equals（和 hashCode）方法通常比手动实现更可取，因为 IDE 不会出现粗心的错误，而人则会。*IDEs, too, have facilities to generate equals and hashCode methods, but the resulting source code is more verbose and less readable than code that uses AutoValue, does not track changes in the class automatically, and therefore requires testing. That said, having IDEs generate equals (and hashCode) methods is generally preferable to implementing them manually because IDEs do not make careless mistakes, and humans do.*

In summary, don’t override the equals method unless you have to: in many cases, the implementation inherited from Object does exactly what you want.If you do override equals, make sure to compare all of the class’s significant fields and to compare them in a manner that preserves all five provisions of the equals contract.

总之，除非必须，否则不要覆盖 equals 方法：在许多情况下，从 Object 继承而来的实现正是你想要的。如果你确实重写了 equals，那么一定要比较类的所有重要字段，并以保持 equals 约定的所有 5 项规定的方式进行比较。*In summary, don't override the equals method unless you have to: in many cases, the implementation inherited from Object does exactly what you want. If you do override equals, make sure to compare all of the class's significant fields and to compare them in a manner that preserves all five **provisions<规定>** of the equals contract.*

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-3/Chapter-3-Introduction.md)**

- **Next Item（下一条目）：[Item 11: Always override hashCode when you override equals（当覆盖 equals 方法时，总要覆盖 hashCode 方法）](/Chapter-3/Chapter-3-Item-11-Always-override-hashCode-when-you-override-equals.md)**
