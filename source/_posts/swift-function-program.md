---
title: 【译】Swift和函数式编程的精髓
date: 2019-10-11 09:58:08
tags:
---

我想说这真的是一篇非常非常好的文章，它通过对一个实例的API的优化，教会我们如何写出优美简洁的Swift的函数式代码。但是这个文章是视频中作者的口述，所以翻译过程中难免有不当之处。大家可以对着视频和原文进行观看和对比。

[原文地址](https://academy.realm.io/posts/tryswift-rob-napier-swift-legacy-functional-programming/)

 ### 介绍
Swift第一次被公布的一周后，我写了一篇名为“[Swift不是函数式](http://robnapier.net/swift-is-not-functional)”的博文。两年后，Swift仍然不是函数式。这篇文章并没有对此进行阐述，而是将讨论函数式语言几十年来的经验和研究，我们可以用自己快速的方式将这些经验和研究带入Swift。

#### 什么是函数式编程
它是单子、仿函数、haskell和优雅的代码。

函数式编程不是一种语言或语法，而是一种思考问题的方式。函数式编程在我们有`单子`之前已经存在了几十年。函数式编程是一种思考如何分解问题，然后以结构化的方式将它们重新组合在一起的方法。

让我们从swift中的一个简单示例开始。

```swift
var persons: [Person] = []
for name in names {
    let person = Person(name: name)
    if person.isValid {
        persons.append(person)
    }
}
```
这是一个简单的循环，它做两件事：将`name`转换成`Person`，然后将这些人放入一个有效的数组中。很简单，但这里有很多事情。为了理解发生了什么，你需要在头脑中执行这个。你需要思考，“这个数组是做什么的？“

它看起来很简单，但可能会更简单。我们可以把问题分开。我们可以把东西拆开。
我们现在有两个循环，每个循环做的更少。每一个都很简单，更简单的代码让我们找到模式：创建一个数组，遍历一些值，对每个值执行一些操作，然后在最后将它们放到另一个数组中。

当你有一些事情做了很多次，你可能应该提取函数-Swift有一个，它被称为`map`。`map`将某物的列表转换为其他某物的列表。重要的是它说明了它的含义：可能`persons`是`names`到`person`的映射。这是一份根据人名列出的名单。我们不必在头脑中执行任何东西，他就在代码中表述我们的意思。

```swift
let possiblePersons = names.map(Person.init)
let persons = possiblePersons.filter { $0.isValid }
```
另一个循环也是一个非常常见的模式：它有一个名为“filter”的方法。filter接受一个谓词，它是一个返回`bool`的函数。它使用函数给我们返回有效的数据。我们可以把这些结合在一起，从而获得`possiblePersons`，然后把他们加入我们的过滤器。

从七行代码到两行代码。另外，我们可以重新组合它们：这些都是我们可以重新组合起来的值。我们用链式把他们进行组合。

它很容易阅读，我们可以一行一行地阅读。

```swift
let persons = names
    .map(Person.init)
    .filter { $0.isValid }
```
它十分易学并且很容易适应这种方式。在这样的例子中你不必再写一个for循环了。
### 函数式工具
1977年，John Backus（协助发明了Fortran 和 Algol）获得了图灵奖，发表演讲“编程能从冯诺依曼风格中解放出来吗？“我喜欢这个标题。“冯·诺依曼风格”是指Fortran和Algol。

这篇论文是他为发明它们而做的声明。他表示命令式编程，一步一步地改变某种状态，直到获得你想要的最终状态。当他说“函数式”的时候，那并不是指我们今天所说的函数式，但他启发了许多函数式研究人员去研究和学习。

这篇论文引起了我的兴趣，我们可以回到swift：我们如何将复杂的事情分解成简单的事情。把这些简单的东西泛化，然后用一些规则把它们粘在一起，比如代数。

代数是一套规则，用来把事物组合在一起，把它们拆开，然后转换它们。我们可以想出可以用来操纵程序的规则。我们已经做到了：我们做了一个循环，我们把它分解成两个简单的循环，找到每一个循环的通用形式，然后使用链式将它们重新组合在一起。当haskell程序员第一次遇到swift时，他们往往会感到沮丧，因为试着做它们在Haskell语言中做的事。

在haskell中，与几乎所有的函数式语言一样，组成的基本单元是函数。有很多漂亮的方法来组合函数。我可以通过将`foldr`函数和`+`函数粘合在一起，并将其初始值设为0，来创建一个名为sum的新函数。不管你读起来是否舒服，只要你这样做了，它就相当漂亮了。

```swift
let sum = foldr (+) 0
  sum [1..10]
```
你可以用swift来做，但是它会很难看，而且它不能很好地工作，因为你在用错误单元组合它们。swift不是函数式。

swift中的组成单位是类型。类、结构、枚举和协议，这些都是可以组合的。我们通常把两种类型粘在一起。通过实现一个函数并将它们粘在一起，我们可以用更简单的片段来构建它们。

swift中另一种非常常见的构图是将类型放在语境中。你最常用的是`optionals`。可选类型是根据语境确定的，在语境中可能有值也可能没有值。这是一个与类型相关的小信息：它是否存在？这就是语境的含义。添加语境比其他跟踪额外信息的方法强大得多。

```swift
extension MyStruct<T>: Sequence {
    func makeIterator() -> AnyIterator<T> {
return ... }
}
```
我们可以跟踪一个事实，即不存在整数，或者根本不存在值，这是因为我们会从-1这样的整数中窃取一个值，这意味着没有值。但是现在你必须到处测试，这是丑陋的，容易出错的，编译器不能帮你。

如果我们将整数改为可选的，这时编译器可以帮助您。你有这样的语境“它存在吗，它不存在吗，我可以帮助你确保你不会忘记这一点。”如果你曾经使用过-1，很容易忘记检查，然后你的程序变得不可控了。

### 例子
让我们建立一个更复杂的例子。

```swift
func login(username: String, password: String,
           completion: (String?, Error?) -> Void)
login(username: "rob", password: "s3cret") {
    (token, error) in
    if let token = token {
        // success
    } else if let error = error {
// failure }
}
```
这是一个非常常见的API。我们有一个带有`username`和`password`参数的`login`函数，在某个时刻，它将返回一个`token`和一个可能的错误。我认为我们可以通过考虑语境做得更好。

**第一个问题** 是这个`completion`。我说它是一个`string`，是什么样的`string`。她是一个`token`.我可以给它贴个标签，但那没用。在swift中，标签不是类型的一部分。即使我这么做了，还是这个`string`。字符串可以是指很多东西。

`Tokens`有规则：例如，它们可能必须是固定长度，或者不能为空。这些都是可以用字符串做的，但不能用`tokens`。我们希望有更多关于`token`的上下文；它有规则，所以我们希望捕获这些规则。我们可以做到，我们可以给它更多的结构。这就是为什么它被称为`struct`。

```swift
struct Token {
    let string: String
}
```
我把字符串放入结构体中。这不会花费任何成本，也不会导致间接或任何额外的内存使用，但现在我可以对此设置规则。

你只能用特定的字符串来构造它们。我可以在上面加上一些扩展，这样就没有必要用任意的字符串了。这要好多了，我可以对所有类型都这样做；字符串当然没问题，也可以是字典、数组和整数类型。

当您拥有这些类型时，您可以将它们提升到上下文中，并控制可以放在它们上的内容。你可以控制他表达的意思。我不需要标签或注释，因为第一个参数显然是一个`token`，因为它的类型是`Token`。

**第二个问题** 是我们要传递`username`和`password`。在大多数有这些的程序中，您总是将它们一起传递；`password`本身尤其无用。我想创建一个规则，允许我用“and”组合用户名和密码，所以我需要一个“and”类型。我们有一个，它又是一个结构。

### "AND"类型 
```swift
struct Credential {
  var username: String
  var password: String
}
```
结构体是"and"类型。 `Credential` 是由`username`和`password`组成.  “and”类型被称为 “生产类型”.
我鼓励你大声说出来。例如：“凭证是用户名和密码。”这有意义吗？如果它没有意义，也许它是错误的类型，或者你创建它是错误的。

```swift
func login(credential: Credential,
           completion: (Token?, Error?) -> Void)


let credential = Credential(username: "rob",
                            password: "s3cret")
login(credential: credential) { (token, error) in
    if let token = token {
        // success
    } else if let error = error {
// failure }
}
```

现在我们可以交换`Credential`，而不是`username`和`password`：这也使得我们的签名更短更清晰。我们还提供了许多不错的可能性：在`Credentials`上添加扩展，在其他类型的规则下交换它们。也许我们想要10次`password`，或者`access token`，或者`facebook`或者`google`等等。现在，我不需要更改代码的任何其他部分，因为我只是传递`credentials`。

不过，它也有问题。我们通过了元组(Token?, Error?) –元组是“and”类型。它们是匿名结构。我们的意思是“也许是token，也许是error”？有四种可能性：两者都有，或者两者都没有，或者一个或另一个。只有两种可能性是有意义的。如果我得到了一个`token`和`error`？这是一个错误条件吗？我需要一个致命的错误吗？我需要忽略它吗？你需要考虑一下这个问题，并可能针对它编写测试。

### “OR” Type (Sum)

问题是你不是说“也许”什么的-你是指一个`token`或一个`error`。我们有可以使用的“或”类型吗？
```swift
enum Result<Value> {
    case success(Value)
    case failure(Error)
}
```
It is an enum - enums are “or” types (this or that), whereas structs are “and” types (this and that). Like “and” types are called “product types”, “or” types are called “sum types”.
这是一个枚举 - `enums`是 “or”类型，就像结构体是“and”类型。正如“and”类型被称作“生产类型”，“or”类型被称为“和类型”

```swift
func login(credential: Credential,
           completion: (Result<Token>) -> Void)
login(credential: credential) { result in
    switch result {
    case .success(let token): // success
    case .failure(let error): // failure
    }
}
```
我想建立这个`result`类型。这让我很困扰，因为它不是内置在swift中的。它很容易建造。我们将提升我们的值，给它更多的语境。它从一种值转变为一个成功的值。

我们的错误变成一个失败的错误，我们有更多的语境。如果我们将`result`、生成的`token`扔进我们的api中，那么我们必须针对所有这些情况编写测试来保证所有不可能的情况都会消失。我们不必担心他们，因为他们是不可能的。我希望错误不可能，而不是编写测试用例。

我喜欢这个API。我用`credential`登录，它会给我一个生成的`token`。

### 这个课程
这是函数式编程的真正精髓，也是我们应该带给swift的：复杂的事情可以分解成更小、更简单的事情。

我们可以为这些简单的事情找到通用的解决方案，我们可以使用一致的规则将这些简单的事情重新组合起来，让我们对我们的程序进行推理。这使得编译器更容易查出bug，我认为70年代的`John Backus`完全同意这一点。把它拆了，把它造起来。
