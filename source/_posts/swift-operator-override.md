---
title: 【译】如何在swift中使用函数式编程
date: 2019-08-28 15:50:28
tags:
---

原文链接：
https://www.raywenderlich.com/4018226-overloading-custom-operators-in-swift
## swift重载自定义运算符

 在本Swift教程中，您将学习如何创建自定义运算符，重载现有运算符以及设置运算符优先级。
 
 运算符是任何编程语言的核心构建模块。 你能想象编程而不使用+或=吗？
 运算符非常基础，大多数语言都将它们作为编译器（或解释器）的一部分进行处理。 但是Swift编译器并不对大多数操作符进行硬编码，而是为库提供了创建的操作符的方法。 它将工作留给了Swift标准库，用来提供您期望的所有常见操作符。 这种微妙的差异为巨大的定制潜力打开了大门。
 
 Swift运算符特别强大，因为您可以通过两种方式更改它们以满足您的需求：为现有运算符分配新功能（称为运算符重载），以及创建新的自定义运算符。
 在本教程中，您将使用一个简单的`Vector`结构并构建自己的一组运算符，以组合不同的向量。

### 入门
打开**Xcode**，然后通过**File▶New▶Playground**创建一个**Playground**。 选择空白模板并命名您的**Playground**为`CustomOperators`。 删除所有默认代码，以便您可以从空白平板开始。

将以下代码添加到您的**Playground**：

``` swift
struct Vector {
  let x: Int
  let y: Int
  let z: Int
}

extension Vector: ExpressibleByArrayLiteral {
  init(arrayLiteral: Int...) {
    assert(arrayLiteral.count == 3, "Must initialize vector with 3 values.")
    self.x = arrayLiteral[0]
    self.y = arrayLiteral[1]
    self.z = arrayLiteral[2]
  }
}

extension Vector: CustomStringConvertible {
  var description: String {
    return "(\(x), \(y), \(z))"
  }
}
```

在这里，您可以定义一个含有三个属性的`Vector`类型，它遵循两个协议。 `CustomStringConvertible`协议和`description`计算属性允许您打印一个友好的字符串用来代表`Vector`。

在`playground`的底部，添加以下代码：

```swift
let vectorA: Vector = [1, 3, 2]
let vectorB = [-2, 5, 1] as Vector
```

在没有初始化器的情况下,你刚刚用了两个`Array`来创建两个`Vector`!那是如何发生的呢?
`ExpressibleByArrayLiteral`协议提供了一个平滑的接口来初始化`Vector`。该协议需要一个具有可变参数的不可用初始化程序：`init（arrayLiteral：Int ...）`

可变参数`arrayLiteral`允许您传入由逗号分隔的无限数量的值。例如，您可以创建`Vector`，像这样`Vector(arrayLiteral：0）`或`Vector（arrayLiteral：5,4,3）`。

该协议进一步方便，并允许您直接使用数组进行初始化，只要您明确定义类型，这是您为`vectorA`和`vectorB`所做的。

这种方法的唯一警告是你必须接受任何长度的数组。如果您将此代码放入应用程序中，请记住，如果传入长度不是三的数组，它将会崩溃。如果您尝试初始化少于或多于三个值的`Vector`，则初始化程序顶部的断言将在开发和内部测试期间在控制台中提醒您。

单独的矢量很好，但如果你能用它们做事情会更好。正如你在学校时所做的那样，你将开始你学习 **加法** 之旅。

### 重载加法运算符
运算符重载的一个简单示例是加法运算符。 如果您将它与两个数字一起使用，则会发生以下情况：
```swift
1 + 1 // 2
```
但是，如果对字符串使用相同的加法运算符，则它具有完全不同的行为：
```swift
"1" + "1" // "11"
```
当+与两个整数一起使用时，它会以算术形式把它们相加。 但是当它与两个字符串一起使用时，它会将它们连接起来。

为了使运算符重载，您必须实现一个名称为运算符符号的函数。

**注意：** 您可以将重载函数定义为类方法，这是您将在本教程中执行的操作。 这样做时，必须将其声明为`static`函数，以便可以在没有定义它的实例的情况下访问它。

在`playground`最后面添加如下代码:
```swift
// MARK: - Operators
extension Vector {
  static func + (left: Vector, right: Vector) -> Vector {
    return [
      left.x + right.x,
      left.y + right.y,
      left.z + right.z
    ]
  }
}

```

此函数将两个向量作为参数，并将它们的和作为新向量返回。 为了让向量相加，只需组成向量的变量相加.

要测试此功能，请将以下内容添加到`playground`的底部：
```swift
vectorA + vectorB // (-1, 8, 3)
```
您可以在`playground`的右侧边栏中看到向量相加的结果.

### 其他类型的运算符
加法运算符是所谓的`infx`运算符，意味着它在两个不同的值之间使用。 
还有其他类型的运算符：

* `infix`：在两个值之间使用，例如加法运算符（例如，1 + 1）
* `prefix`：在值之前添加，如负号运算符（例如-3）。
* `postfix`：在值之后添加，比如强制解包运算符（例如，mayBeNil！）
* `ternary`：在三个值之间插入两个符号。 在Swift中，不支持用户自定义的三元运算符，只有一个内置的三元运算符，您可以在[Apple的文档中阅读](https://docs.swift.org/swift-book/LanguageGuide/BasicOperators.html#ID71)。
您要重载的下一个运算符是负号，它将更改`Vector`的每个变量的符号。 例如，如果将它应用于`vectorA`，即`（1,3,2）`，则返回`（-1，-3，-2）`。


在这个`extension`中的前面一个`static`函数下面添加如下代码:
```swift
static prefix func - (vector: Vector) -> Vector {
  return [-vector.x, -vector.y, -vector.z]
}
```
运算符默认是`infix`类型，如果您希望运算符是其他类型，则需要在函数声明中指定运算符类型。 负号运算符不是`infix`类型，因此您将`prefix`修饰符添加到函数声明中。

在`playground`的底部，添加以下代码：
```swift
-vectorA // (-1, -3, -2)
```
在侧栏中检查结果是否正确。

接下来是减法，我将留给你实现自己。 完成后，请检查以确保您的代码与我的代码类似。 提示：减法与加上一个负数相同。

试一试，如果您需要帮助，请查看下面的解决方案！
```swift
static func - (left: Vector, right: Vector) -> Vector {
  return left + -right
}
```
通过添加已下代码到`playground`底部,测试你的新操作符的输出:
```swift
vectorA - vectorB // (3, -2, 1)
```

### 不同类型的参数？ 没问题！
您还可以通过标量乘法将向量乘以数字。 要将向量乘以2，可以将每个分量乘以2。 你接下来就要实现这个.

您需要考虑的一件事是参数的顺序。 当你实现加法的时候，顺序无关紧要，因为两个参数都是向量。

对于标量乘法，您需要考虑`Int * Vector`和`Vector * Int`。 如果您只实现其中一种情况，Swift编译器将不会自动知道您希望它以其他顺序工作。

要实现标量乘法，请在刚刚添加的减法函数下添加以下两个函数：
```swift
static func * (left: Int, right: Vector) -> Vector {
  return [
    right.x * left,
    right.y * left,
    right.z * left
  ]
}

//这里用到了上面重载的*运算
static func * (left: Vector, right: Int) -> Vector {
  return right * left
}
```

为避免多次写入相同的代码，第二个函数只是将其参数转发给第一个。

在数学中，向量有另一个有趣的操作，称为叉乘。 叉乘超出了本教程的范围，但您可以在[Cross product Wikipedia](https://en.wikipedia.org/wiki/Cross_product)页面上了解有关它们的更多信息。

由于在大多数情况下不鼓励使用自定义符号（谁想在编码时打开表情符号菜单？），重复使用星号进行叉乘操作会非常方便。

与标量乘法不同，叉乘将两个向量作为参数并返回一个新向量。

添加以下代码以在刚刚添加的乘法函数之后实现叉乘：
```swift
static func * (left: Vector, right: Vector) -> Vector {
  return [
    left.y * right.z - left.z * right.y,
    left.z * right.x - left.x * right.z,
    left.x * right.y - left.y * right.x
  ]
}

```
现在，将以下计算添加到`playground`的底部，同时利用乘法和叉乘运算符：
```swift
vectorA * 2 * vectorB // (-14, -10, 22)
```
此代码找到`vectorA`和2的标量乘法，然后找到该向量与`vectorB`的交叉乘积。 请注意，星号运算符始终从左向右，因此前面的代码与使用括号分组操作相同，如`（vectorA * 2）* vectorB`。

### 运算符协议

某些协议需要实现一些运算符符。 例如，符合`Equatable`的类型必须实现`==`运算符。 类似的，符合`Comparable`的类型必须至少实现`<`和`==`，因为`Comparable`继承自`Equatable`。 `Comparable`类型也可以选择实现>，>=和<=，但这些运算符具有默认实现。

对于`Vector`，`Comparable`并没有太多意义，但`Equatable`有意义，因为如果它们的组件全部相等，则两个向量相等。 接下来你将实现`Equatable`。

去实现协议，请在`playground`的末尾添加以下代码：
```swift
extension Vector: Equatable {
  static func == (left: Vector, right: Vector) -> Bool {
    return left.x == right.x && left.y == right.y && left.z == right.z
  }
}
```
添加以下代码在`palyground`的底部,并且测试他的输出
```swift
vectorA == vectorB // false
```
上述代码返回`false`因为`vectorA`和`vectorB`有不同的组件.
实现`Equatable`协议,不仅能够检查这些类型的相等性。 您还获得了自由访问`contains(_:)`的矢量数组！

### 创建自定义运算符

请记住我们通常不鼓励使用自定义运算符. 当然该规则也有例外。

关于自定义符号的一个好的经验法则是，只有在满足以下条件时才应使用它们：

* 它们的含义是众所周知的，或者对阅读代码的人有意义。
* 它们很容易在键盘上打出来.
最后一个你要实现的运算符符合这两个条件。 矢量点积运算两个向量并返回单个标量数。 您的运算符会将向量中的每个值乘以另一个向量中的对应值，然后将所有这些乘积相加。
点积的符号为`•`，您可以使用键盘上的`Option-8`轻松键入。
您可能会想，“我可以在本教程中对其他所有操作符执行相同的操作，对吧？”

不幸的是，你还不能那样做。 在其他情况下，您重载已存在的运算符。 对于新的自定义运算符，您需要首先创建运算符。

直接在Vector实现最下面，但在`CustomStringConvertible`扩展之上，添加以下声明：
```swift
infix operator •: AdditionPrecedence
```
将`•`定义为放在两个其他值之间的运算符，并且与加法运算符`+`具有相同的优先级。 暂时忽略优先级，因为你会在学到它。

既然已经注册了此运算符，请在运算符扩展的末尾添加其实现，紧接在运算符`*`的实现的下面：
```swift
static func • (left: Vector, right: Vector) -> Int {
  return left.x * right.x + left.y * right.y + left.z * right.z
}
```
添加下面代码到`playground`测试他的输出:
```swift
vectorA • vectorB // 15
```
到目前为止，一切看起来都不错......或者是吗？ 在`playground`的底部尝试以下代码：
```swift
vectorA • vectorB + vectorA // Error!
```
现在，`•`和`+`具有相同的优先级，因此编译器从左到右解析表达式。 编译器将您的代码解释为：
```swift 
(vectorA • vectorB) + vectorA
```
此表达式归结为`Int + Vector`，您尚未实现并且不打算实现。 你要如何来解决这个问题？

### 优先级组(Precedence Groups)
Swift中的所有运算符都属于一个优先级组，它描述了运算符的运算顺序。 还记得学习小学数学中的操作顺序吗？ 这基本上就是你在这里所要处理的。
在Swift标准库中，优先顺序如下：
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/8/28/16cd7213aaf59301~tplv-t2oaga2asx-image.image))

以下是关于这些运算符的一些注释，您之前可能从来没有看到过它们：

1. 位移运算符`<<`和`>>`用于二进制计算。
2. 您使用转换运算符`is`和`as`来确定或更改值的类型。
3. `nil`合并运算符,`??`为可选类型提供默认值。
4. 如果您的自定义运算符未指定优先级，则会自动分配`DefaultPrecedence`。
5. 三元运算符，`？ ：`，类似于if-else语句。
6. `AssignmentPrecedence`,衍生出来的`=`，在所有运算之后进行运算。
编译器解析类型具有左关联性，所以`v1 + v2 + v3 ==（v1 + v2）+ v3`。 

操作符按它们在表中出现的顺序进行解析。 尝试使用括号重写以下代码：
```swift
v1 + v2 * v3 / v4 * v5 == v6 - v7 / v8
```
当您准备好检查数学时，请查看下面的解决方案。
```swift
(v1 + (((v2 * v3) / v4) * v5)) == (v6 - (v7 / v8))
```
在大多数情况下，您需要添加括号以使代码更易于阅读。 无论哪种方式，理解编译器评估运算符的顺序都很有用。

### Dot运算符优先级
你新定义的`dot-product`不适合任何这些类别。 它的优先级必须小于加号运算符（如前所述），但它是否真的适合`CastingPrecedence`或`RangeFormationPrecedence`？

取而代之，您将为您的点积运算符创建自己的优先级组。

用以下内容替换`•`运算符的原始声明：
```swift
precedencegroup DotProductPrecedence {
  lowerThan: AdditionPrecedence
  associativity: left
}

infix operator •: DotProductPrecedence
```
在这里，您创建一个新的优先级组并将其命名为`DotProductPrecedence`。 您将它放在低于`AdditionPrecedence`的位置，因为您希望加号运算符优先。 你也可以将它设为左关联，因为你想要从左到右进行评估，就像加法和乘法中一样。 然后，将此新优先级组分配给`•`运算符。

 **注意**:除了`lowerThan`之外，您还可以在`DotProductPrecedence`中指定`higherThan`。 如果您在单个项目中有多个自定义优先级组，这一点就变得很重要。
 你之前写的代码返回了你希望的结果:
 ```swift
vectorA • vectorB + vectorA // 29
```
**恭喜💐**--你掌握了自定义运算符

### 接下来你可以做些什么
你可以阅读本教程完整的代码,代码已经在最后面给出.
此时，您知道如何根据需要定义Swift操作符。 在本教程中，您专注于在数学领域中使用运算符。 在实践中，您将找到更多使用运算符的方法。

在[ReactiveSwift](https://github.com/ReactiveCocoa/ReactiveSwift)框架中可以看到自定义运算符使用的一个很好的演示。 一个例子是`<~`用来绑定，这是响应式编程中的一个重要功能。 以下是此运算符的使用示例：
```swift
let (signal, _) = Signal<Int, Never>.pipe()
let property = MutableProperty(0)
property.producer.startWithValues {
  print("Property received \($0)")
}

property <~ signal
```
[Cartography](https://github.com/robb/Cartography)是另一个大量使用运算符重载的框架。 `AutoLayout`工具重载相等和比较运算符，以使`NSLayoutConstraint`创建更简单：

```swift
constrain(view1, view2) { view1, view2 in
  view1.width   == (view1.superview!.width - 50) * 0.5
  view2.width   == view1.width - 50
  view1.height  == 40
  view2.height  == view1.height
  view1.centerX == view1.superview!.centerX
  view2.centerX == view1.centerX

  view1.top >= view1.superview!.top + 20
  view2.top == view1.bottom + 20
}
```
此外，您始终可以参考官方文档[ custom operator documentation](https://docs.swift.org/swift-book/LanguageGuide/AdvancedOperators.html)。

有了这些新的灵感来源，您可以走出世界，通过运算符重载使代码更简单。 小心不要对自定义运算符太迷恋！：]


```swift
extension Vector: ExpressibleByArrayLiteral {
  init(arrayLiteral: Int...) {
    assert(arrayLiteral.count == 3, "Must initialize vector with 3 values.")
    self.x = arrayLiteral[0]
    self.y = arrayLiteral[1]
    self.z = arrayLiteral[2]
  }
}

precedencegroup DotProductPrecedence {
  lowerThan: AdditionPrecedence
  associativity: left
}

infix operator •: DotProductPrecedence

extension Vector: CustomStringConvertible {
  var description: String {
    return "(\(x), \(y), \(z))"
  }
}

let vectorA: Vector = [1, 3, 2]
let vectorB: Vector = [-2, 5, 1]

// MARK: - Operators
extension Vector {
  static func + (left: Vector, right: Vector) -> Vector {
    return [
      left.x + right.x,
      left.y + right.y,
      left.z + right.z
    ]
  }
  
  static prefix func - (vector: Vector) -> Vector {
    return [-vector.x, -vector.y, -vector.z]
  }
  
  static func - (left: Vector, right: Vector) -> Vector {
    return left + -right
  }
  
  static func * (left: Int, right: Vector) -> Vector {
    return [
      right.x * left,
      right.y * left,
      right.z * left
    ]
  }
  
  static func * (left: Vector, right: Int) -> Vector {
    return right * left
  }
  
  static func * (left: Vector, right: Vector) -> Vector {
    return [
      left.y * right.z - left.z * right.y,
      left.z * right.x - left.x * right.z,
      left.x * right.y - left.y * right.x
    ]
  }
  
  static func • (left: Vector, right: Vector) -> Int {
    return left.x * right.x + left.y * right.y + left.z * right.z
  }
}

vectorA + vectorB // (-1, 8, 3)
-vectorA // (-1, -3, -2)
vectorA - vectorB // (3, -2, 1)

extension Vector: Equatable {
  static func == (left: Vector, right: Vector) -> Bool {
    return left.x == right.x && left.y == right.y && left.z == right.z
  }
}

vectorA == vectorB // false

vectorA • vectorB // 15

vectorA • vectorB + vectorA // 29

```


