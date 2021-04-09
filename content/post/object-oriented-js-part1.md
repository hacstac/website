---
title: "Object Oriented JavaScript In a Easy Way [Part-1]"
date: 2020-01-27T20:00:00+08:00
tags: ["Javascript", "Programming"]
categories: ["Javascript"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### What is Object Oriented Programming In JavaScript?

Object Oriented Programming is a popular style of programming which has been holding the roots of JavaScript since the beginning.
If you already have prior experience with Object Oriented Programming in another language, please put aside the knowledge you have, and read through the entire module with a beginner's mind.
This is because Object Oriented Programming in JavaScript is very different from Object Oriented Programming in Other Languages. It is more difficult than it really is.

<!--more-->

Now, lets begin by finding out what OOP's in JavaScript is about :

---

## Creating objects from a common object :

Objects in JavaScript have `properties` and `methods`. These objects can resemble real-life things. here, we are trying to construct a `user`  object in JavaScript. A `user` has a `firstName`, `lastName` and an `age` . We could add these attributes as properties in JavaScript.

User has some details and we can extract these details through function `getDetails()` , we can write this function as a `method` in the `user` object as well.

{{< gist AkashRajvanshi aeea808cf37aaf83784017019b5c2e05 "commonObject.js" >}}

By using this method, for every new user, we have to write this whole code again which will make the code long and messy.

Instead of using the above method, we can create a function(or something similar) which will make individuals.

Then construct a function that has a `this` keyword, and we will create individuals with `new` keyword.

{{< gist AkashRajvanshi 9969907cb40fbcac41d2a34154bf0c87 "constructorFunctionExample.js" >}}

Note: In Object Oriented Programming, the first letter of the constructor is capitalized (`User`), while each instance is written like a normal variable ( akash, abha etc..).

This small differentiation instantly shows you the difference between constructors and instances in your code.

---

## What's "this"?

`this` is a keyword in JavaScript. When it is used in a constructor, it refers to the instance that is created with the constructor.

`this` is a very important concept in Object Oriented Programming. So you need to be familiar with it.

Unfortunately, the value of `this` in JavaScript changes depending how you call a function, which is unexpected, and can cause a lot of confusion.

So, you'll learn more about `this` in the given lesson to help you familiarize with the different possible values `this` can take up.

[How To Master JavaScript's "this" Keyword!!](https://blogs.infusionstack.tech/post/javascript-this-keyword/)

---

## There are 4 layer of oops in js :

- Layer 1 - Object Orientation with single objects
- Layer 2 - Prototype Chains of Objects
- Layer 3 - Constructors as factories for instances, similar to classes in other languages
- Layer 4 - Sub classing, creating new constructors by inheriting from existing ones

---

## These 4 Layers in Nutshell :

- Layer 1: Single objects - how do objects, JavaScript's basic OOP building blocks, work in isolation?
- Layer 2: Prototype Chains: Each Object has a chain of zero or more prototype objects. Prototypes are JavaScript's core inheritance mechanism.
- Layer 3 : Classes - JavaScript's classes are factories for objects. The relationship between a class and its instance is based on prototypal inheritance.
- Layer 4 : Sub classing - The relationship between a subclass and its super class is also based on prototypal inheritance.

---

## Layer 1 : Object Orientation with Single Objects

### What is Object?

An object in JavaScript is a data type which is composed of a collection of `names` or `keys` and `values`, represented in a `key` : `value` pairs. The `key` : `value` pairs consist of properties that may contain any data type — including strings, numbers, and Boolean's — as well as methods, which are functions contained within an object.

### What are the Properties, Keys & Values?

All objects in JavaScript are maps(dictionaries) from strings to values. an entry in an object is called a `Property`. The `key` of property is always a text string and the `value` of a property can be any JavaScript value, including a function. Methods are properties whose values are functions.

{{< gist AkashRajvanshi 989bcae04e983688fb118eab54f6f2b3 "propertyKeysExample.js" >}}

### There are three kinds of properties :

- Properties: These are normal properties in an object, that is mapping from string keys to values. & 1 more is included call as `named data properties` which include `methods`
- Accessors: Special methods whose invocations look like reading or writing properties.(normal properties are storage locations for property values, accessors allow us to compute the values of properties).
- Internal Properties: These are not directly accessible from JavaScript, but there might be some indirect ways of accessing them :

    Example : `[[Prototype]]`holds the prototype of an object and is readable via `Object.getPrototypeof()`.


### Two Roles of objects in JavaScript :

- Object as Records
- Object as Dictionaries

----

#### Object as Records :

- Records : Objects-as-records have a fixed number of properties, whose keys are known at development time. Their values can have different types.

Object as records are created via so-called object-literals. object literals is a stand out feature of JavaScript: which allows us to directly create object without a need for classes.

Example :

{{< gist AkashRajvanshi b37732b423bf0122b44c8d0c5e0846a3 "objectAsRecords.js" >}}

##### Dot Operator (.): Accessing Properties via Fixed Keys:

- Getting Properties : Dot operator allows us "get" a property (read its value).

{{< gist AkashRajvanshi 4aeff8490581112f14acc0b4e64a8e7a "dotOperatorGettingProp.js" >}}


- Calling Methods : Dot operator is also used for call methods.

{{< gist AkashRajvanshi 0275f3e53d62c5aca64bc2ec725e8131 "dotOperatorCallMethod.js" >}}


- Setting Properties : For setting up values, we use assignment operator (=) via dot notation.

{{< gist AkashRajvanshi 0485239fb42f388a98b7426594242c6e "dotOperatorSettingMethod.js" >}}


- Deleting Properties : The `delete` operator allows us to completely remove a property (The whole key-value pair).

{{< gist AkashRajvanshi fd762ae4c88f4cd6493846c9d3328939 "dotOperatorDeleteMethod1.js" >}}


`delete` returns `false` , if the property is an own property which cannot be deleted. otherwise returns `true` in all other cases.

{{< gist AkashRajvanshi 8b305388f152227bd71864b8a6a2f1c5 "dotOperatorDeleteMethod2.js" >}}


`delete` returns true even if it doesn't change anything (inherited properties are never removed)

```JavaScript
    delete user.toString // true
    console.log(user.toString) // [λ: toString] (Still Exists)
```

Accessors : There are two kinds of accessors in JavaScript :

1.  Getter (reading) a property.
2.  Setter (writing) a property.


- Getter : A getter is created by prefixing a method definition with the keyword `get` :

{{< gist AkashRajvanshi ab1935f5254e24f9df7d5e2d066f02e1 "dotOperatorGetterMethod.js" >}}

- Setter : A setter is created by prefixing a method definition with the keyword `set` :

{{< gist AkashRajvanshi a29880a18eca7c78751a9cf18685286d "dotOperatorSetterMethod.js" >}}

##### Spreading into object Literals (...)

Inside an object literal, a spread property adds the properties of an another object to the current one :

{{< gist AkashRajvanshi 2f695bd24956daf23c79668d0c88ee42 "spreadOperatorwithDot.js" >}}

##### Methods :

We know that the ordinary functions play several roles. `Method` is one of those roles.

Example :

{{< gist AkashRajvanshi a9a14ed5a559214474256b2f2db13b8e "methodsInObject.js" >}}

- Call : `call` is a function that allows us to call/invoke/activate another function. When we use `call` to call a function, we can change the `this` context.

Example :

{{< gist AkashRajvanshi e97dbaa55c59db3cb6f29a79bb8b3529 "callMethod.js" >}}


`call` is used to borrow methods from another function (or method).

Example :
Let's say you have an array of numbers. You want to use `slice` to copy the numbers into a new array. One way to use `slice` is to call the slice method through the numbers array.

Another way through `call` Is when you use `call`, you need to pass the numbers array as the `this` context.

{{< gist AkashRajvanshi d80264a1ae740f5eddffa7239b2090b7 "callMethod1.js" >}}

But why we use this `Array.prototype.slice.call` over `array.slice` when its so complicated?

People use `call` with array methods because most array methods (like `slice`, `forEach`, `filter`, and `map`) work with both arrays and array-like object.

Example : We can also do operations on array-like object.

```JavaScript
    const obj = { foo: [1, 2, 3] }
    const arrayMethods = Array.prototype.reverse.call(obj.foo)
    console.log(arrayMethods) // [3, 2, 1]
```

- Apply()

The only difference between call and apply method is the way argument is passed. In case of `apply` method, second argument is an `array` of arguments where as in case of `call` method, arguments are passed individually.

{{< gist AkashRajvanshi f40ac110488e74c329f8fe3ee23797f5 "applyMethod.js" >}}

Note : With ES6, there's no need for `apply`  Method because we can spread arrays with the `spread` operator.

- Bind()

`bind` allows you to change the `this` context as well. However, instead of invoking the function immediately, `bind` returns a function with the parameters you have provided.

{{< gist AkashRajvanshi 845996709e3716b78bfdf0c4f86e1f94 "bindMethod.js" >}}

##### Unusual Property keys :

We can't use reserved words (such as var and function) as a variable names, but we can use them as property keys :

```JavaScript
    var obj = { var: 'a', function: 'b' };
    obj.var //'a'
    obj.function //'b'
```

---

### Object as Dictionaries :

Dictionaries: Objects-as-dictionaries have a variable number of properties, whose keys are not known at development time. All of their values have the same type.

Object work best as records. But before ES6, JavaScript did not have a data structure for dictionaries (ES6 brought Maps). Therefore, objects had to be used as dictionaries. As a consequence, `keys` had to be strings, but `values` could have arbitrary types.

Arbitrary fixed strings as property keys :

As we move forward from object-as-records to object-as-dictionaries,  There will be an important change is that we must be able to use `arbitrary strings & numbers` as property keys.

{{< gist AkashRajvanshi 72d31f60bd717c095ed26f545d6ae726 "objAsDictionaries.js" >}}

#### Bracket Operator ([ ]): Accessing Properties via Computed Keys

Bracket Operator allow us to refer to a property via an expression.

- Getting Properties

{{< gist AkashRajvanshi c03699055d008dff35ff7a8bf470c8f4 "bracketOperatorGettingProp.js" >}}

- Calling methods

{{< gist AkashRajvanshi 25a3b0e5b6455e5d77347fd939707c62 "bracketOperatorCallingMethod.js" >}}

- Setting Properties

{{< gist AkashRajvanshi 2a91ed0af07098dc17614561e1ab1f33 "bracketOperatorSettingMethod.js" >}}

- Deleting Properties

{{< gist AkashRajvanshi c570975d841ff55e5fb088128c11cd0a "bracketOperatorDeleteMethod.js" >}}

#### Detecting Properties :

We will use the `in` operator to detect the properties because `in` operator checks if the given key exists in the hash table and it considered as the best way to determine whether the property exists in an object. Also, for performance wise it won't delay.

In some cases, we might want to check for the existence of a property only if it is an own property. `in` operator checks for both own properties and prototype properties. So we will need to take a different approach `hasOwnProperty()`  function, if returns `true` when the property owns by the object, and returns `false` for prototype property.

In given example 'toString' is a prototype property.

{{< gist AkashRajvanshi d092299bba7497cabf5e06c9db9e1916 "DetectingProperties.js" >}}

---

## Advanced Methods :

### Standard Methods :

`Object.prototype` defines several standard methods which can be overridden. Two important ones are :

-  `.toString`: configures how objects are converted to strings :  Mainly it  can be useful for debugging purposes

{{< gist AkashRajvanshi d9276499159241454ce3506b24dd6e82 "toStringMethod.js" >}}

-  `.valueOf`: configures how objects are converted to numbers

{{< gist AkashRajvanshi 584cca459838110f69bc7fce08ea3aff "valueOfMethod.js" >}}

### Enumeration :

By default, all properties that you add to an object are enumerable, which means that you can iterate over them using a `for-in` loop. Enumerable properties have their internal `[[Enumerable]]` attributes set to true. The `for-in` loop enumerates all enumerable properties on an object.

You can easily understand this with the help of given example :

{{< gist AkashRajvanshi 00b7ae5dd30c4ef14ff80b1ef0faab3d"enumeration1.js" >}}

If you want to list all of an object's properties to use later in your program. ECMAScript 5 Introduced the `Object.keys ()` method to retrieve an array of enumerable property names.

Note: Not all the properties are enumerable.

In fact, most of the native methods on objects have their `[[Enumerable]]` attribute set to `false`. You can check whether a property is enumerable by using the `propertyIsEnumerable()` method, which is present on every object:

{{< gist AkashRajvanshi d9c296f16a50e715ee4395d405f69bf6 "enumeration2.js" >}}

### Property Attributes & Property Descriptors

Property Attributes : Property attributes are the atomic building block of properties.

 All of a property's state, both its data and its metadata, is stored in attributes. They are fields that a property has, much like an object has properties.

- Attributes keys are often written in double brackets.
- Attributes matter for normal properties and for accessors.

Property Descriptor : It is a data structure for working programmatically with attributes.

Common Attributes :

   - Enumerable : Determines whether you can iterate over the property or not.
   - Configurable : Determines whether the property can be changed or not.

1.  Data Property Attributes : ( Contains Extra Two )

- Value : Which holds the property value ( You can understand it better in example )
- Writable : Which is a Boolean value indicating whether the property can be written or not.

{{< gist AkashRajvanshi a7a196d6d810c5cac6cdb332db9f37d6 "dataPropertyAttributes.js" >}}

1. Accessor Property Attributes ( Contains Different Two )

Accessors properties also have two additional attributes. As there is no value stored for accessor properties, that's why it is no need for `[[value]]` or `[[writable]]`.

- Get : Contains Getter Function
- Set : Contains Setter Function

{{< gist AkashRajvanshi c19219767f9c6aef78936043d4580301 "accessorPropertiesAttributes.js" >}}

1. Multiple Properties

It's also possible to define multiple properties on an object simultaneously if you use `Object.defineProperties()` instead of `Object.defineProperty()`

{{< gist AkashRajvanshi 92a2d529a17e743a30b26a93abd4aad8 "multipleProperties.js" >}}

1. Retrieval Method

If you want to fetch property attributes, you can easily do with using `Object.getOwnPropertyDescriptor().`

- `Object.getOwnPropertyDescriptor()` : Returns an object with enumerable, configurable, writable, value

{{< gist AkashRajvanshi 4afc4add816c0ee6bcc417f2cab46758 "retrivealMethod.js" >}}

### Preventing Object Modification : To Making objects Immutable

As we know that objects are by default mutable means we can add new properties to it but to prevent these additions we have attribute called `[[Extensible]]`, which is Boolean in nature.

if `[[Extensible]]` is `false` then we can prevent new properties from being added to an object.

By default, `[[Extensible]]` is set to be a `true` on every object

There are also three different ways to preventing objects from modification :

1. Preventing Extensions
2. Sealing Objects
3. Freezing Objects

### 1. Preventing Extensions

In this method, we can use `Object.preventExtensions()` method. After this we'll never be able to add any new properties to it again.

We can also check the value of `[[Extensible]]` by using method `Object.isExtensible()`

Example :

{{< gist AkashRajvanshi 9ec154fbcee88a8eccfb54707ca285d1 "PreventingExtensions.js" >}}

### 2. Sealing objects

In this method, we can use `Object.seal()` method and this function `seal` the object which means that we neither only add properties nor can remove or change their types.

Example :

{{< gist AkashRajvanshi 46028cfa83bb8e0cca0e3a67f9794f3d "sealingObject.js" >}}

### 3. Freezing Objects

In this method we can use `Object.freeze()` method and this function `freezes` the object. which is  similar to sealing function but in this method we can't also write the existing properties.

Data properties in this method are read only.

Note : frozen object can't become unfrozen

Example :

{{< gist AkashRajvanshi e648ed89851204bd627b0cb9753d7677 "freezingObjects.js" >}}


---
