---
title: 【译】如何在swift中使用函数式编程
date: 2019-10-03 21:50:34
tags:
---

翻译：`https://www.raywenderlich.com/9222-an-introduction-to-functional-programming-in-swift#toc-anchor-012`

在本教程中，您将逐步学习如何开始使用函数式编程以及如何编写声明性代码而不是命令式代码。

swift于2014年在WWDC上进入编程世界的大门，它不仅仅是一门新的编程语言。 它为iOS和macOS平台的软件开发提供了便利。

本教程重点介绍其中一种方法：函数式编程，简称FP。 您将了解FP中使用的各种方法和技术。

### 开始
创建一个新的`playground`通过选择`File ▸ New ▸ Playground`
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/3/16d9244e23c78c87~tplv-t2oaga2asx-image.image)
设置你的`playground`，通过拖拽分割线你可以看到结果面板和控制台
![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/10/3/16d924565d497ebb~tplv-t2oaga2asx-image.image)

现在删除`playground`中所有代码，添加一下行：

```swift
import Foundation
```
开始在大脑中回忆一些基础理论吧。

### 命令式编程风格
当你第一次学习编码时，你可能学会了命令式的风格。 命令式风格如何运作？
添加下面代码到你的`playground`:
```swift
var thing = 3
//some stuff
thing = 4
```
该代码是正常和合理的。 首先，你创建一个名为`thing`的变量等于3，然后你命令`thing`变为4。

简而言之,这就是命令式的风格。 您使用一些数据创建变量，然后将该变量改为其他数据。

### 函数式编程概念
在本节中，您将了解FP中的一些关键概念。 许多论文表明**immutable state(状态不变)**和**lack of side effects(没有副作用)**是函数式编程两个最重要的特征，所以你将先学习它们。

### 不变性和副作用
无论您首先学习哪种编程语言，您可能学到的最初概念之一是变量代表数据或状态。 如果你退一步思考这个想法，变量看起来很奇怪。

术语“变量”表示随程序运行而变化的数量。 从数学角度思考数量`thing`，您已经将**时间**作为软件运行方式的关键参数。 通过更改变量，可以创建`mutable state`(可变状态)。

要进行演示，请将此代码添加到您的`playground`：
```swift
func superHero() {
  print("I'm batman")
  thing = 5
}

print("original state = \(thing)")
superHero()
print("mutated state = \(thing)")
```
神秘变化！为什么`thing`变成5了？这种变化被称为** side effect **。函数`superHero（）`更改了一个它自己没有定义的变量。

单独或在简单系统中，可变状态不一定是问题。将许多对象连接在一起时会出现问题，例如在大型面向对象系统中。可变状态可能会让人很难理解变量的值以及该值随时间的变化。

例如，在为多线程系统编写代码时，如果两个或多个线程同时访问同一个变量，它们可能会无序地修改或访问它。这会导致意外行为。这种意外行为包括竞争条件，死锁和许多其他问题。

想象一下，如果你可以编写状态永远不会发生变化的代码。并发系统中出现的一大堆问题将会消失。像这样工作的系统具有不可变状态，这意味着不允许状态在程序的过程中发生变化。

使用不可变数据的主要好处是使用它的代码单元没有副作用。代码中的函数不会改变自身之外的元素，并且在发生函数调用时不会出现任何怪异的效果。您的程序可以预测，因为没有副作用，您可以轻松地重现其预期的效果。

本教程涵盖了高级的FP编程，因此在现实世界中考虑概念是有帮助的。在这种情况下，假设您正在构建一个游乐园的应用程序，并且该游乐园的后端服务器通过REST API提供数据。

### 创建游乐园的模型
通过添加以下代码到`playground`去创建数据结构
```swift 
enum RideCategory: String, CustomStringConvertible {
  case family
  case kids
  case thrill
  case scary
  case relaxing
  case water

  var description: String {
    return rawValue
  }
}

typealias Minutes = Double
struct Ride: CustomStringConvertible {
  let name: String
  let categories: Set<RideCategory>
  let waitTime: Minutes

  var description: String {
    return "Ride –\"\(name)\", wait: \(waitTime) mins, " +
      "categories: \(categories)\n"
  }
}
```
接着通过model创建一些数据：
```swift
let parkRides = [
  Ride(name: "Raging Rapids",
       categories: [.family, .thrill, .water],
       waitTime: 45.0),
  Ride(name: "Crazy Funhouse", categories: [.family], waitTime: 10.0),
  Ride(name: "Spinning Tea Cups", categories: [.kids], waitTime: 15.0),
  Ride(name: "Spooky Hollow", categories: [.scary], waitTime: 30.0),
  Ride(name: "Thunder Coaster",
       categories: [.family, .thrill],
       waitTime: 60.0),
  Ride(name: "Grand Carousel", categories: [.family, .kids], waitTime: 15.0),
  Ride(name: "Bumper Boats", categories: [.family, .water], waitTime: 25.0),
  Ride(name: "Mountain Railroad",
       categories: [.family, .relaxing],
       waitTime: 0.0)
]
```

当你声明`parkRides`通过`let`代替`var`,数组和它的内容都不可变了。
尝试通过下面代码修改数组中的一个单元：
```swift
parkRides[0] = Ride(name: "Functional Programming",
                    categories: [.thrill], waitTime: 5.0)
```
产生了一个编译错误，是个好结果。你希望Swift编译器阻止你改变数据。
现在删除错误的代码继续教程。

### 模块化
使用模块化就像玩儿童积木一样。 你有一盒简单的积木，可以通过将它们连接在一起来构建一个庞大而复杂的系统。 每块砖都有一份工作，您希望您的代码具有相同的效果。

假设您需要一个按字母顺序排列的所有游乐设施名称列表。 从命令性地开始这样做，这意味着利用可变状态。 将以下功能添加到`playground`的底部：
```swift
func sortedNamesImp(of rides: [Ride]) -> [String] {

  // 1
  var sortedRides = rides
  var key: Ride

  // 2
  for i in (0..<sortedRides.count) {
    key = sortedRides[i]

    // 3
    for j in stride(from: i, to: -1, by: -1) {
      if key.name.localizedCompare(sortedRides[j].name) == .orderedAscending {
        sortedRides.remove(at: j + 1)
        sortedRides.insert(key, at: j)
      }
    }
  }

  // 4
  var sortedNames: [String] = []
  for ride in sortedRides {
    sortedNames.append(ride.name)
  }

  return sortedNames
}

let sortedNames1 = sortedNamesImp(of: parkRides)
```
你的代码完成了以下工作：
1. 创建一个变量保存排序的`rides`
2. 遍历传入函数的`rides`
3. 使用插入排序排序`rides`
4. 遍历排序的`rides`获得名称

添加下面代码到`playground`验证函数是否按照意图执行：
```swift
func testSortedNames(_ names: [String]) {
  let expected = ["Bumper Boats",
                  "Crazy Funhouse",
                  "Grand Carousel",
                  "Mountain Railroad",
                  "Raging Rapids",
                  "Spinning Tea Cups",
                  "Spooky Hollow",
                  "Thunder Coaster"]
  assert(names == expected)
  print("✅ test sorted names = PASS\n-")
}

print(sortedNames1)
testSortedNames(sortedNames1)
```
现在你知道如果将来你改变排序的方式（例如：使其函数式），你可以检测到任何发生的错误。
从调用者到` sortedNamesImp(of:)`的角度看，他提供了一系列的`rieds`,然后输出按照名字排序的列表。` sortedNamesImp(of:)`之外的任何东西都没有改变。
你可以用另一个测试证明这点，将下面代码添加到`playground`底部：

```swift
var originalNames: [String] = []
for ride in parkRides {
  originalNames.append(ride.name)
}

func testOriginalNameOrder(_ names: [String]) {
  let expected = ["Raging Rapids",
                  "Crazy Funhouse",
                  "Spinning Tea Cups",
                  "Spooky Hollow",
                  "Thunder Coaster",
                  "Grand Carousel",
                  "Bumper Boats",
                  "Mountain Railroad"]
  assert(names == expected)
  print("✅ test original name order = PASS\n-")
}

print(originalNames)
testOriginalNameOrder(originalNames)
```
在这个测试中，你将收集作为参数传递的游乐设施列表的名称，并根据预期的顺序测试该订单。
在结果区和控制台中，你将看到` sortedNamesImp(of:)`内的排序`rides`不会影响输入列表。你创建的模块化功能是半函数式的。按照名称排序`rides`是逻辑单一，可以测试的，模块化的并且可重复利的函数。
` sortedNamesImp(of:)`中的命令式代码用于长而笨重的函数。该功能难以阅读，你无法轻易知道他干了什么事情。在下一部分你将学习如何进一步简化` sortedNamesImp(of:)`等函数中的代码。

### 一等和高阶函数
在FP语言中，函数式一等公民。你可以把函数当成对象那样那样进行赋值。
因此，函数可以接收其他函数作为参数或者返回值。接受或者返回其他函数的函数成为高阶函数。
在本节中，你将使用FP语言中的三种常见的高阶函数：`filter`,`map`,`reduce`.

#### Filter
在swift中，`filter`是`Collection`类型的方法，例如Swift数组。它接受另一个函数作为参数。此另一个函数接受来自数组的单个值作为输入，检查该值是否属于并返回`Bool`.
`filter`将输入函数应用于调用数组的每个元素并返回另一个数组。输出函数仅包含参数函数返回true的数组元素。
试试下面的例子：
```swift
let apples = ["🍎", "🍏", "🍎", "🍏", "🍏"]
let greenapples = apples.filter { $0 == "🍏"}
print(greenapples)
```
在输入数组中有三个青苹果，你将看到输出数组中含有三个青苹果。
回想一下你用`sortedNamesImp(of:) `干了什么事情。
1. 遍历所有的`rides`传递给函数的。
2. 通过名字排序`rides`
3. 获取已排序的`riedes`的名字

不要过分的考虑这一点，而是以声明的方式思考它，即考虑你想要发生什么而不是如何发生。首先创建一个函数，该函数将`Ride`对象作为函数的输入参数：
```swift
func waitTimeIsShort(_ ride: Ride) -> Bool {
  return ride.waitTime < 15.0
}
```

这个函数`waitTimeIsShort(_:)`接收一个`Ride`，如果`ride`的等待时间小于15min返回true，否则返回false。
`parkRides`调用`filter`并且传入刚刚创建的函数。
```swift
let shortWaitTimeRides = parkRides.filter(waitTimeIsShort)
print("rides with a short wait time:\n\(shortWaitTimeRides)")
```
在`playground`输出中，你只能在调用`filter(_:)`的输出中看到` Crazy Funhouse`和` Mountain Railroad`,这是正确的。
由于swift函数也被叫闭包，因此可以通过将尾随闭包传递给过滤器并且使用闭包语法来生成相同的结果：
```swift
let shortWaitTimeRides2 = parkRides.filter { $0.waitTime < 15.0 }
print(shortWaitTimeRides2)
```
这里，`filter(_:)`让`$0`代表了`parkRides`中的每个`ride`，查看他的`waitTime`属性并且测试它小于15min.你声明性的告诉程序你希望做什么。在你使用的前几次你会觉得这样很神秘。

#### Map
集合方法`map（_:）`接受单个函数作为参数。在将该函数应用于集合的每个元素之后，它输出一个相同长度的数组。映射函数的返回类型不必与集合元素的类型相同。

试试这个：
```swift
let oranges = apples.map { _ in "🍊" }
print(oranges)
```
你把每一个苹果都映射成一个橘子，制作一个橘子盛宴。
您可以将`map（_:）`应用于`parkrides`数组的元素，以获取所有`ride`名称的字符串列表：
```swift
let rideNames = parkRides.map { $0.name }
print(rideNames)
testOriginalNameOrder(rideNames)
```
您已经证明了使用`map（_:）`获取`ride`名称与在集合使用迭代操作相同，就像您之前所做的那样。
当你使用集合类型上`sorted（by:）`方法执行排序时，也可以按如下方式排序`ride`的名称：
```swift
print(rideNames.sorted(by: <))
```
集合方法`sorted（by:）`接受一个比较两个元素并返回bool作为参数的函数。因为运算符`<`是一个牛逼的函数，所以可以使用swift缩写的尾随闭包{$0<$1}。swift默认提供左侧和右侧。

现在，您可以将提取和排序`ride`名称的代码减少到只有两行，这要感谢`map（:）`和`sorted（by:）`。
使用以下代码将`sortedNamesImp(_:)`重新实现为`sortedNamesFP(_:)`：
```swift
func sortedNamesFP(_ rides: [Ride]) -> [String] {
  let rideNames = parkRides.map { $0.name }
  return rideNames.sorted(by: <)
}

let sortedNames2 = sortedNamesFP(parkRides)
testSortedNames(sortedNames2)
```
你的声明性代码更容易阅读，你可以轻松地理解它是如何工作的。测试证明`sortedNamesFP(_:) `和`sortedNamesImp(_:).`做了相同的事情。

#### Reduce
集合方法`reduce（::）`接受两个参数：第一个是任意类型T的起始值，第二个是一个函数，该函数将同一T类型的值与集合中的元素组合在一起，以生成另一个T类型的值。
输入函数一个接一个地应用于调用集合的每个元素，直到它到达集合的末尾并生成最终的累积值。
例如，您可以将这些桔子还原为一些果汁：
```swift
let juice = oranges.reduce("") { juice, orange in juice + "🍹"}
print("fresh 🍊 juice is served – \(juice)")
```
从空字符串开始。然后为每个桔子的字符串添加`🍹`。这段代码可以为任何数组注入果汁，因此请小心放入它：]。
为了更实际，添加以下方法，让您知道公园中所有游乐设施的总等待时间。
```swift
let totalWaitTime = parkRides.reduce(0.0) { (total, ride) in 
  total + ride.waitTime 
}
print("total wait time for all rides = \(totalWaitTime) minutes")
```
此函数的工作方式是将起始值0.0传递到`reduce`中，并使用尾随闭包语法来添加每次骑行占用的总等待时间。代码再次使用swift简写来省略return关键字。默认情况下，返回`total+ride.waittime`的结果。
在本例中，迭代如下：
```swift
Iteration    initial    ride.waitTime    resulting total
    1          0            45            0 + 45 = 45
    2         45            10            45 + 10 = 55
    …
    8        200             0            200 + 0 = 200
```
如您所见，得到的总数将作为下一次迭代的初始值。这将一直持续，直到`reduce`迭代了`parkRides`中的每个`Ride`。这允许你用一行代码得到总数！

### 先进技术
您已经了解了一些常见的FP方法。现在是时候用更多的函数理论来做进一步的研究了。
#### Partial Functions（局部函数）
部分函数允许您将一个函数封装到另一个函数中。要了解其工作原理，请将以下方法添加到playground：
```swift
func filter(for category: RideCategory) -> ([Ride]) -> [Ride] {
  return { rides in
    rides.filter { $0.categories.contains(category) }
  }
}
```
这里，`filter（for:）`接受一个`ridecategory`作为其参数，并返回一个类型为`（[Ride]）->[Ride]`的函数。输出函数接受一个`Ride`对象数组，并返回一个由提供的`category`过滤的`Ride`对象数组。

在这里通过寻找适合小孩子的游乐设施来检查过滤器：
```swift
let kidRideFilter = filter(for: .kids)
print("some good rides for kids are:\n\(kidRideFilter(parkRides))")
```
您应该可以在控制台输出中看到`Spinning Tea Cups`和`Grand Carousel`。

#### 纯函数

FP中的一个主要概念是纯函数，它允许您对程序结构以及测试程序结果进行推理。
如果函数满足两个条件，则它是纯函数：

* 当给定相同的输入时，函数总是产生相同的输出，例如，输出仅取决于其输入。
* 函数在其外部没有副作用。

在playground中添加以下纯函数：
```swift
func ridesWithWaitTimeUnder(_ waitTime: Minutes, 
    from rides: [Ride]) -> [Ride] {
  return rides.filter { $0.waitTime < waitTime }
}
```
`rides withwaittimeunder（_:from:）`是一个纯函数，因为当给定相同的等待时间和相同的`rides`列表时，它的输出总是相同的。

有了纯函数，就很容易针对该函数编写一个好的单元测试。将以下测试添加到您的playgroud：
```swift
let shortWaitRides = ridesWithWaitTimeUnder(15, from: parkRides)

func testShortWaitRides(_ testFilter:(Minutes, [Ride]) -> [Ride]) {
  let limit = Minutes(15)
  let result = testFilter(limit, parkRides)
  print("rides with wait less than 15 minutes:\n\(result)")
  let names = result.map { $0.name }.sorted(by: <)
  let expected = ["Crazy Funhouse",
                  "Mountain Railroad"]
  assert(names == expected)
  print("✅ test rides with wait time under 15 = PASS\n-")
}

testShortWaitRides(ridesWithWaitTimeUnder(_:from:))
```
请注意你是如何将`ridesWithWaitTimeUnder(_:from:)`传递给测试。请记住，函数是一等公民，您可以像传递任何其他数据一样传递它们。这将在下一节派上用场。
另外，运行你的测试程序再次使用`map（_:）`和`sorted（_by:）`提取名称。你在用FP测试你的FP技能。

### 参照透明度

纯函数与参照透明的概念有关。如果一个程序的元素可以用它的定义替换它，并且总是产生相同的结果，那么它的引用是透明的。它生成可预测的代码，并允许编译器执行优化。纯函数满足这个条件。

通过将函数体传递给` ridesWithWaitTimeUnder(_:from:)`，可以验证函数` testShortWaitRides(_:)`是否具有引用透明性：
```swift
testShortWaitRides({ waitTime, rides in
    return rides.filter{ $0.waitTime < waitTime }
})
```

在这段代码中，你获取了`ridesWithWaitTimeUnder(_:from:)`，并将其直接传递给封装在闭包语法中的`testShortWaitrides（:）`。这证明了`ridesWithWaitTimeUnder(_:from:) `是引用透明的。

在重构某些代码时,希望确保不会破坏任何东西，引用透明性是很有用。引用透明代码不仅易于测试，而且还允许您在不必验证实现的情况下移动代码。

### 递归

最后要讨论的概念是递归。每当函数调用自身作为其函数体的一部分时，都会发生递归。在函数式语言中，递归替换了许多在命令式语言中使用的循环结构。

当函数的输入导致函数调用自身时，就有了递归情况。为了避免函数调用的无限堆栈，递归函数需要一个基本情况来结束它们。



您将为您的`rides`添加一个递归排序函数。首先，使用下面的拓展让`Ride`遵循`Comparable`协议：
```swift
extension Ride: Comparable {
  public static func <(lhs: Ride, rhs: Ride) -> Bool {
    return lhs.waitTime < rhs.waitTime
  }

  public static func ==(lhs: Ride, rhs: Ride) -> Bool {
    return lhs.name == rhs.name
  }
}
```
在这个扩展中，可以使用运算符重载来创建允许比较两个`rides`的函数。您还可以看到在排序之前使用的<运算符的完整函数声明`sorted(by:)`。
如果等待时间更少，那么一个`ride`就少于另一个`ride`，如果`rides`具有相同的名称，则`rides`是相等的。
现在，扩展数组以包含`quickSorted`方法：
```swift
extension Array where Element: Comparable {
  func quickSorted() -> [Element] {
    if self.count > 1 {
      let (pivot, remaining) = (self[0], dropFirst())
      let lhs = remaining.filter { $0 <= pivot }
      let rhs = remaining.filter { $0 > pivot }
      return lhs.quickSorted() + [pivot] + rhs.quickSorted()
    }
    return self
  }
}
```
此扩展允许您对数组进行排序，只要元素是可比较的。
快速排序算法首先选择一个基准元素。然后将集合分成两部分。一部分包含小于或等于基准元素的所有元素，另一部分包含大于基准元素的其余元素。然后使用递归对这两部分进行排序。注意，通过使用递归，您不需要使用可变状态。

输入以下代码以验证您的方法是否正常工作：
```swift
let quickSortedRides = parkRides.quickSorted()
print("\(quickSortedRides)")


func testSortedByWaitRides(_ rides: [Ride]) {
  let expected = rides.sorted(by:  { $0.waitTime < $1.waitTime })
  assert(rides == expected, "unexpected order")
  print("✅ test sorted by wait time = PASS\n-")
}

testSortedByWaitRides(quickSortedRides)
```
在这里，您将检查您的解决方案是否与来自受信任的swift标准库函数的预期值匹配。
请记住递归函数具有额外的内存使用和运行时开销。在数据集变得更大之前，您不必担心这些问题。

### 命令与声明性代码风格

在本节中，您将结合您所学到的关于FP的知识来清楚地演示函数编程的好处。
考虑以下情况：
一个有小孩的家庭希望在频繁的浴室休息之间尽可能多地乘车。他们需要找出哪一种适合儿童乘车的路线最短。帮助他们找出所有家庭乘坐等待时间少于20分钟，并排序他们最短到最长的等待时间。

#### 用命令式方法解决问题
考虑一下如何用强制算法来解决这个问题。试着用你自己的方法解决这个问题。
您的解决方案可能类似于：
```swift
var ridesOfInterest: [Ride] = []
for ride in parkRides where ride.waitTime < 20 {
  for category in ride.categories where category == .family {
    ridesOfInterest.append(ride)
    break
  }
}

let sortedRidesOfInterest1 = ridesOfInterest.quickSorted()
print(sortedRidesOfInterest1)
```
把这个加到你的`playground`上并执行它。你应该看到，`Mountain Railroad`, `Crazy Funhouse` 和`Grand Carousel` 是最好的乘坐选择，该名单是为了增加等待时间。

正如所写的，命令式代码很好，但快速浏览并不能清楚地显示它正在做什么。你必须停下来仔细看看算法来掌握它。当您六个月后返回进行维护时，或者将代码交给新的开发人员时，代码是否容易理解？

添加此测试以将FP方法与您的命令式解决方案进行比较：
```swift
func testSortedRidesOfInterest(_ rides: [Ride]) {
  let names = rides.map { $0.name }.sorted(by: <)
  let expected = ["Crazy Funhouse",
                  "Grand Carousel",
                  "Mountain Railroad"]
  assert(names == expected)
  print("✅ test rides of interest = PASS\n-")
}

testSortedRidesOfInterest(sortedRidesOfInterest1)
```

#### 用函数方法解决问题

使用FP解决方案，您可以使代码更具自解释性。将以下代码添加到您的playground：
```swift
let sortedRidesOfInterest2 = parkRides
    .filter { $0.categories.contains(.family) && $0.waitTime < 20 }
    .sorted(by: <)
```
通过添加以下内容，验证这行代码是否生成与命令代码相同的输出：
```swift
testSortedRidesOfInterest(sortedRidesOfInterest2)
```
在一行代码中，您告诉swift要计算什么。您希望将您的`parkRides`过滤到具有小于20分钟的等待时间的`.family`的游乐设施，然后对它们排序。这就彻底解决了上述问题。
生成的代码是声明性的，这意味着它是自解释的，并且读起来就像它解决的问题陈述。
这与命令式代码不同，命令式代码读起来像是计算机解决问题语句所必须采取的步骤。

### 函数编程的时间和原因

Swift不是纯粹的函数式编程语言，但它结合了多种编程范式，为您提供了应用程序开发的灵活性。
开始使用FP技术的一个好地方是在模型层和应用程序的业务逻辑出现的地方。您已经看到创建这种逻辑的离散测试是多么容易。
对于用户界面，不太清楚看哪里可以使用FP编程。`Reactive programming`是一种用于用户界面开发的类似于FP的方法的例子。例如，RxSwift是一个用于IOS和MACOS编程的反应库。
通过使用函数式，声明性方法，代码变得更加简洁明了。另外，当代码被隔离到没有副作用的模块化函数中时，它将更容易测试。
当你想最大化你的多核CPU的全部潜力时，最小化并发带来的副作用和问题是很重要的。FP是一个很好的工具，在你的技能中应对那些问题。

本文涉及的全部代码：
```swift
/// Copyright (c) 2018 Razeware LLC
///
/// Permission is hereby granted, free of charge, to any person obtaining a copy
/// of this software and associated documentation files (the "Software"), to deal
/// in the Software without restriction, including without limitation the rights
/// to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
/// copies of the Software, and to permit persons to whom the Software is
/// furnished to do so, subject to the following conditions:
///
/// The above copyright notice and this permission notice shall be included in
/// all copies or substantial portions of the Software.
///
/// Notwithstanding the foregoing, you may not use, copy, modify, merge, publish,
/// distribute, sublicense, create a derivative work, and/or sell copies of the
/// Software in any work that is designed, intended, or marketed for pedagogical or
/// instructional purposes related to programming, coding, application development,
/// or information technology.  Permission for such use, copying, modification,
/// merger, publication, distribution, sublicensing, creation of derivative works,
/// or sale is expressly withheld.
///
/// THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
/// IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
/// FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
/// AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
/// LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
/// OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
/// THE SOFTWARE.

import Foundation

//: # Introduction to Functional Programming

/*:
 ## Imperative Style
 
 Command your data!
 */
var thing = 3
//some stuff
thing = 4

/*:
 ## Side effects
 
 Holy mysterious change! - Why is my thing now 5?
 */
func superHero() {
  print("I'm batman")
  thing = 5
}

print("original state = \(thing)")
superHero()
print("mutated state = \(thing)")

/*:
 ## Create a Model
 */
enum RideCategory: String {
  case family
  case kids
  case thrill
  case scary
  case relaxing
  case water
}



typealias Minutes = Double
struct Ride {
  let name: String
  let categories: Set<RideCategory>
  let waitTime: Minutes
}


/*:
 ## Create some data using that model
 */
let parkRides = [
  Ride(name: "Raging Rapids",
       categories: [.family, .thrill, .water],
       waitTime: 45.0),
  Ride(name: "Crazy Funhouse", categories: [.family], waitTime: 10.0),
  Ride(name: "Spinning Tea Cups", categories: [.kids], waitTime: 15.0),
  Ride(name: "Spooky Hollow", categories: [.scary], waitTime: 30.0),
  Ride(name: "Thunder Coaster",
       categories: [.family, .thrill],
       waitTime: 60.0),
  Ride(name: "Grand Carousel", categories: [.family, .kids], waitTime: 15.0),
  Ride(name: "Bumper Boats", categories: [.family, .water], waitTime: 25.0),
  Ride(name: "Mountain Railroad",
       categories: [.family, .relaxing],
       waitTime: 0.0)
]

/*:
 ### Attempt to change immutable data.
 */

//parkRides[0] = Ride(name: "Functional Programming", categories: [.thrill], waitTime: 5.0)

/*:
 ## Modularity
 
 Create a function that does one thing.
 
 1. Returns the names of the rides in alphabetical order.
 */

func sortedNamesImp(of rides: [Ride]) -> [String] {
  
  // 1
  var sortedRides = rides
  var key: Ride
  
  // 2
  for i in (0..<sortedRides.count) {
    key = sortedRides[i]
    
    // 3
    for j in stride(from: i, to: -1, by: -1) {
      if key.name.localizedCompare(sortedRides[j].name) == .orderedAscending {
        sortedRides.remove(at: j + 1)
        sortedRides.insert(key, at: j)
      }
    }
  }
  
  // 4
  var sortedNames: [String] = []
  for ride in sortedRides {
    sortedNames.append(ride.name)
  }
  
  return sortedNames
}

let sortedNames1 = sortedNamesImp(of: parkRides)

//: Test your new function
func testSortedNames(_ names: [String]) {
  let expected = ["Bumper Boats",
                  "Crazy Funhouse",
                  "Grand Carousel",
                  "Mountain Railroad",
                  "Raging Rapids",
                  "Spinning Tea Cups",
                  "Spooky Hollow",
                  "Thunder Coaster"]
  assert(names == expected)
  print("✅ test sorted names = PASS\n-")
}

print(sortedNames1)
testSortedNames(sortedNames1)

var originalNames: [String] = []
for ride in parkRides {
  originalNames.append(ride.name)
}

//: Test that original data is untouched

func testOriginalNameOrder(_ names: [String]) {
  let expected = ["Raging Rapids",
                  "Crazy Funhouse",
                  "Spinning Tea Cups",
                  "Spooky Hollow",
                  "Thunder Coaster",
                  "Grand Carousel",
                  "Bumper Boats",
                  "Mountain Railroad"]
  assert(names == expected)
  print("✅ test original name order = PASS\n-")
}

print(originalNames)
testOriginalNameOrder(originalNames)

/*:
 ## First class and higher order functions.
 
 Most languages that support FP will have the functions `filter`, `map` & `reduce`.
 
 ### Filter
 
 Filter takes the input `Collection` and filters it according to the function you provide.
 
 Here's a simple example.
 */

let apples = ["🍎", "🍏", "🍎", "🍏", "🍏"]
let greenapples = apples.filter { $0 == "🍏"}
print(greenapples)


//: Next, try filtering your ride data
func waitTimeIsShort(_ ride: Ride) -> Bool {
  return ride.waitTime < 15.0
}

let shortWaitTimeRides = parkRides.filter(waitTimeIsShort)
print("rides with a short wait time:\n\(shortWaitTimeRides)")

let shortWaitTimeRides2 = parkRides.filter { $0.waitTime < 15.0 }
print(shortWaitTimeRides2)

/*:
 ### Minor detour: CustomStringConvertible
 
 You want to make your console output look nice.
 */
extension RideCategory: CustomStringConvertible {
  var description: String {
    return rawValue
  }
}

extension Ride: CustomStringConvertible {
  var description: String {
    return "Ride –\"\(name)\", wait: \(waitTime) mins, categories: \(categories)\n"
  }
}

/*:
 ### Map
 
 Map converts each `Element` in the input `Collection` into a new thing based on the function that you provide.
 
 First create oranges from apples.
 */
let oranges = apples.map { _ in "🍊" }
print(oranges)

//: Now extract the names of your rides
let rideNames = parkRides.map { $0.name }
print(rideNames)
testOriginalNameOrder(rideNames)

print(rideNames.sorted(by: <))

func sortedNamesFP(_ rides: [Ride]) -> [String] {
  let rideNames = parkRides.map { $0.name }
  return rideNames.sorted(by: <)
}

let sortedNames2 = sortedNamesFP(parkRides)
testSortedNames(sortedNames2)

/*:
 ### Reduce
 
 Reduce iterates across the input `Collection` to reduce it to a single value.
 
 You can squish your oranges into one juicy string.
 */
let juice = oranges.reduce(""){juice, orange in juice + "🍹"}
print("fresh 🍊 juice is served – \(juice)")

//: Here you **reduce** the collection to a single value of type `Minutes` (a.k.a `Double`)
let totalWaitTime = parkRides.reduce(0.0) { (total, ride) in
  total + ride.waitTime
}
print("total wait time for all rides = \(totalWaitTime) minutes")


/*:
 ## Partial Functions
 
 A function can return a function.
 
 `filter(for:)` returns a function of type `([Ride]) -> ([Ride])`
 it takes and returns an array of `Ride` objects
 */
func filter(for category: RideCategory) -> ([Ride]) -> [Ride] {
  return { (rides: [Ride]) in
    rides.filter { $0.categories.contains(category) }
  }
}

//: you can use it to filter the list for all rides that are suitable for kids.
let kidRideFilter = filter(for: .kids)
print("some good rides for kids are:\n\(kidRideFilter(parkRides))")


/*:
 ## Pure Functions
 
 - Always give same output for same input
 - Have no side effects
 */
func ridesWithWaitTimeUnder(_ waitTime: Minutes,
                            from rides: [Ride]) -> [Ride] {
  return rides.filter { $0.waitTime < waitTime }
}

let shortWaitRides = ridesWithWaitTimeUnder(15, from: parkRides)

func testShortWaitRides(_ testFilter:(Minutes, [Ride]) -> [Ride]) {
  let limit = Minutes(15)
  let result = testFilter(limit, parkRides)
  print("rides with wait less than 15 minutes:\n\(result)")
  let names = result.map{ $0.name }.sorted(by: <)
  let expected = ["Crazy Funhouse",
                  "Mountain Railroad"]
  assert(names == expected)
  print("✅ test rides with wait time under 15 = PASS\n-")
}


testShortWaitRides(ridesWithWaitTimeUnder(_:from:))

//: when you replace the function with its body, you expect the same result
testShortWaitRides({ waitTime, rides in
  rides.filter{ $0.waitTime < waitTime }
})

/*:
 ## Recursion
 
 Recursion is when a function calls itself as part of its function body.
 
 Make `Ride` conform to `Comparable` so you can compare two `Ride` objects:
 */
extension Ride: Comparable {
  static func <(lhs: Ride, rhs: Ride) -> Bool {
    return lhs.waitTime < rhs.waitTime
  }
  
  static func ==(lhs: Ride, rhs: Ride) -> Bool {
    return lhs.name == rhs.name
  }
}

/*:
 Next add a `quickSorted` algorithim to `Array`
 */
extension Array where Element: Comparable {
  func quickSorted() -> [Element] {
    if self.count > 1 {
      let (pivot, remaining) = (self[0], dropFirst())
      let lhs = remaining.filter { $0 <= pivot }
      let rhs = remaining.filter { $0 > pivot }
      return lhs.quickSorted() + [pivot] + rhs.quickSorted()
    }
    return self
  }
}

//: test your algorithm
let quickSortedRides = parkRides.quickSorted()
print("\(quickSortedRides)")


/*:
 check that your solution matches the expected value from the standard library function
 */
func testSortedByWaitRides(_ rides: [Ride]) {
  let expected = rides.sorted(by:  { $0.waitTime < $1.waitTime })
  assert(rides == expected, "unexpected order")
  print("✅ test sorted by wait time = PASS\n-")
}

testSortedByWaitRides(quickSortedRides)

/*:
 ## Imperative vs Declarative style
 
 ### Imperitive style. Fill a container with the right things.
 */
var ridesOfInterest: [Ride] = []
for ride in parkRides where ride.waitTime < 20 {
  for category in ride.categories where category == .family {
    ridesOfInterest.append(ride)
    break
  }
}

let sortedRidesOfInterest1 = ridesOfInterest.quickSorted()
print(sortedRidesOfInterest1)

func testSortedRidesOfInterest(_ rides: [Ride]) {
  let names = rides.map({ $0.name }).sorted(by: <)
  let expected = ["Crazy Funhouse",
                  "Grand Carousel",
                  "Mountain Railroad"]
  assert(names == expected)
  print("✅ test rides of interest = PASS\n-")
}

testSortedRidesOfInterest(sortedRidesOfInterest1)

/*:
 ### Functional Approach
 
 Declare what you're doing. Filter, Sort, Profit :]
 */
let sortedRidesOfInterest2 = parkRides
  .filter { $0.categories.contains(.family) && $0.waitTime < 20 }
  .sorted(by: <)

testSortedRidesOfInterest(sortedRidesOfInterest2)


```