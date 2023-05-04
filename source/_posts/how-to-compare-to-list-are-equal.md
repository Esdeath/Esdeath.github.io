---
title: Flutter如何比较两个list是否相等
date: 2023-01-04 10:38:18
tags:
---
在 Flutter 中，你可以使用 Dart 的内置方法 listEquals 来比较两个列表是否相等。listEquals 方法会比较两个列表的长度和元素，如果长度相等且每个元素都相等（使用 == 运算符比较），那么这两个列表就被认为是相等的。

```
List<int> list1 = [1, 2, 3];
List<int> list2 = [1, 2, 3];
List<int> list3 = [1, 2, 4];

bool areList1AndList2Equal = listEquals(list1, list2); // 返回 true
bool areList1AndList3Equal = listEquals(list1, list3); // 返回 false

```
请注意，`listEquals` 方法会使用`==` 运算符比较列表元素，因此对于自定义对象，你可能需要在类中重写 `==` 运算符和 `hashCode` 方法，以便正确地进行比较。
在 `Dart` 中，如果要比较两个自定义对象，通常需要重写 `==` 运算符和 `hashCode` 方法。这是因为默认情况下，`==` 运算符仅比较两个对象的引用，而不是它们的内容。`hashCode` 方法则用于在集合（如 `HashSet` 或 `HashMap`）中标识对象。

以下是一个自定义对象 Person 的示例，演示如何重写 `==` 运算符和 `hashCode` 方法：
```
class Person {
  final String name;
  final int age;

  Person(this.name, this.age);

  // 重写 == 运算符
  @override
  bool operator ==(Object other) {
    // 判断是否为同一类型
    if (other is! Person) {
      return false;
    }

    // 比较内容
    return name == other.name && age == other.age;
  }

  // 重写 hashCode 方法
  @override
  int get hashCode {
    // 使用 bitwise XOR 运算符将各属性的 hashCode 组合在一起
    return name.hashCode ^ age.hashCode;
  }
}

```

现在，你可以正确地比较两个 Person 对象：

```
Person person1 = Person("Alice", 30);
Person person2 = Person("Alice", 30);
Person person3 = Person("Bob", 25);

bool arePerson1AndPerson2Equal = person1 == person2; // 返回 true
bool arePerson1AndPerson3Equal = person1 == person3; // 返回 false

```
请注意，重写 `==` 运算符和 `hashCode` 方法时，需要确保它们的实现是一致的。即，如果两个对象的 `==` 运算符返回 `true`，那么它们的 `hashCode` 也应该相同。这是因为 `hashCode` 用于在集合中标识对象，如果不保持一致性，可能会导致意外的行为。
