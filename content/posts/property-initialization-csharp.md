---
title: "Property Initialization in C#"
date: 2022-06-29T12:00:00+02:00
draft: false
---

[Properties](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/properties) in C# is a very convinient way to write code. A property can be initialized to a default value with the following:

```csharp
public MyClass Item { get; } = new MyClass();
```
The `Item` property is initialized with a new instance of `MyClass` each time we access this property we will get the same instance of the class.

However when writing the following code:

```csharp
public MyClass Item => new MyClass();
```
In that case the `Item` property is not initialized with any value, instead each time we access the `Item` property, the getter will return a new instance of `MyClass`. It can be easily verifiable with the following.

```csharp
public object ItemA { get; } = new object();
public object ItemB => new object();

Debug.Assert(ItemA == ItemA); // true
Debug.Assert(ItemB == ItemB); // false - failure 
```

If we modify the object returned by an initialized property, it will modify the initialized instance of this property, and impact any other reference of it, like in the example below:

```csharp
public MyClass Item { get; } = new MyClass();

var itemB = Item;

Item.member = 'test'

Console.Output(itemB.member) // 'test'
```

When doing this, you want to understand the lifetime of the object you are sharing and also how can other people can modify it.
