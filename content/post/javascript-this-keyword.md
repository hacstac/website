---
title: "How To Master JavaScript's this Keyword!!"
date: 2019-12-27T20:00:00+08:00
tags: ["Javascript", "Programming"]
categories: ["Javascript"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### What is "this" keyword in JavaScript?

`this` is a keyword that's used a lot in Object Oriented Programming. In traditional Object Oriented Programming languages, `this` points to the object. But in JavaScript, the value of `this` changes depending on how we call a function, so `this` can be quite confusing for JavaScript beginners.

<!--more-->

#### So basically what is `this` keyword?

`This` keyword = It's a special identifier keyword that's automatically defined in the scope of every function. which links to the object in which the function operates.

---

## Taking Deep Dive with `This`

- `this` is not a compile time binding it is a run time binding. which dynamically refers to different things in different contexts.

- `this` binding has nothing to do with where a function is declared but instead has everything to do with the manner in which the function is called, also called as **call-site** ( The location in code where a function is called === Call Site ). So all the game of `this` keyword is based on where the call-site is present because the reference of `this` keyword is depend on call site.

- When a function is invoked, an activation record(otherwise known as an execution context) is created. This record contains information about where the function was called from(call-site), how the function was invoked, what parameters were passed, etc.. one of the properties of this record is the `this reference` , which will be used for duration of function's execution.

So now we know what is `this` keyword and which parameters it depends on.

Now we look on the values it works with :

---

## It can take up to four different values:

1. **window**
2. **The object**
3. **The `this` value in its immediate context.**
4. **The listening element.**

---
### 1. `window` Object

- By default `this` refers to global object
- The meaning of `global` object is the default state for the execution context for an execution is `global`, which means if a code is being executed as part of a simple function call then `this` refers to `global` object.

 1. In Browser Environment = `window` object is a global object .
 2. In Node.js Environment = Special object `global` will be the value of this.

 `this` in global scope/simple_function/IIFE === Points to the window object.

- Case 1: `this` in global context.

```JavaScript
 console.log(this) //window
```
![this in global context](https://cdn.hashnode.com/res/hashnode/image/upload/v1569464866845/TCGeG-57H.png)

- Case 2: `this` in simple function.

{{< gist AkashRajvanshi 0698f37d5510b4638fd2b5d02dbc9492 >}}

- Case 3: `this` in  IIFE(Immediately invoked Function).

{{< gist AkashRajvanshi 7903c7c212c3bf7cf8d8a1e8e0f447a9 >}}

- Case 4: `this` in strict mode.

 If strict mode is enabled for any function then the value of `this` will be `undefined` as in strict mode, global object refers to `undefined` in the place of `window` object.

{{< gist AkashRajvanshi d3d31b8228888df30dbcb5004e07c20a >}}

Note: You are working with Web-Pack then you will also see `undefined` as it doesn't allow us to access the global object.

---
### 2. The Object

- Case 1:  "this" in Constructor Function.

`this` in Constructor Function === Points to Newly Created Instance.

When a function is invoked with “new” keyword then the function is known as constructor function and returns a new instance. In such cases, the value of `this` refers to newly created instance.

{{< gist AkashRajvanshi 8459e545b2ab22b57ec381caeeeba2ba  >}}

- Case 2: `this` in Object's Method.

`this` in Object's Method === Points to invoker object (Parent object)

In JavaScript, property of an object can be a method or a simple value. When an Object’s method is invoked then `this` refers to the object which contains the method being invoked.

{{< gist AkashRajvanshi 808b715726bad5f1dc2aaea5d2846f2b >}}

One frustrating thing for beginners to understand is this – `this` always point to `window`, even if the simple function is used in a method.

{{< gist AkashRajvanshi 4c32d09d86d35b0a58b98dcbcfcfa607 >}}

When it is being called as a simple function call then `this` refers to `global object` and when the same definition is invoked as an object’s method then `this` refers to the `parent object`. So the value of `this` depends on how a method is being invoked as well.

---
### 3. The `this` value in its immediate context

`this` in Arrow functions === Points to same `this` value in surrounding scope.

- Case 1 : `this` in Arrow Functions.

`this` in an arrow function always points to the same `this` value in the surrounding scope.

In the example below, the `this` value within `firstName` is the same value as `this` in `name` . Since the `this` value in `name` is the `Person`, `this` within the arrow function also points back to the `Person`.

{{< gist AkashRajvanshi d7a72eaf9ed501ce4131daa3aa6a08ca >}}

When arrow functions are used to create functions in methods, this will point back to the object.

If you use the arrow function to create simple functions in a global context, `this` points to `window` because `window` is the `this` value of the surrounding scope.

{{< gist AkashRajvanshi 659eed1880fcbd848c7317e817ac8699 >}}

---
### 4. The Listing Element

`this` in listing element === Points to the listening element

- Case 1 :  `this` in listening element.

{{< gist AkashRajvanshi 95892e37df1d89ff0711a028d1a09aea >}}

When `this` is used in an event listener, this points back to the button

Remember, when you write event listeners with arrow functions, you can still get the listening element with `event.currentTarget`, even though `this` points to something else.

{{< gist AkashRajvanshi a132641fd07a3b85cb38af879fdaba46  >}}

___

## Binding Rules for `this` Keyword!!

- Default Binding
- Implicit Binding
- Explicit Binding
- `new` Binding
- Lexical binding
- `window` binding

### 1. Default Binding :

Default binding is nothing but a normal calling method of function.

{{< gist AkashRajvanshi 6eb11a325111f5a212cdad8df680da18 >}}

---
### 2. Implicit Binding :

In Implicit binding, `this` keyword will bind with the object which stands before the dot operator.

Implicit binding occurs when dot notation is used to invoke function.

In Implicit binding, we use the object to call the functions with the dot operator.

In Below Code :
`obj` is the `this` for the `foo()` call

{{< gist AkashRajvanshi 4bceddd80164b5cd4a1d1451d862b9cd >}}

---
### 3. Explicit Binding :

In Explicit Binding, we can force a function call to use a particular object for `this` binding, without putting a property function reference on the object. So we can explicitly say to a function what object it should use for `this` — using functions such as `call()`, `apply()` and `bind()`.

Explicit binding of `this` occurs when `.call()`, `.apply()` or `.bind()` are used on a function.

We call these explicit because we can explicitly passing in a `this` content to `call()` or `apply()`

{{< gist AkashRajvanshi 8973061951d9a371b76f972774434b60 >}}

- Case 1: `this` with `call()`, `apply()` methods.

A function in JavaScript is also a special type of object. Every function has call(), bind() and apply() methods. These methods can be used to set custom value of `this` to the execution context of function.

`call()` : The call method calls/invoke/activate a another function with a given `this` value and arguments provided individually.

Syntax  for `call()` function: `sampleFunction.call(thisContext, param1, param2, ... )`

{{< gist AkashRajvanshi ce8e82d2f1a0a48666546146c943e161 >}}

The only difference between `call` and `apply` method is the way argument is passed. In case of `apply` method, second argument is an `array` of arguments where in case of `call` method, arguments are passed individually.

Syntax for `apply()` function: `sampleFunction.apply(thisContext, [param1, param2, ...])`

{{< gist AkashRajvanshi 71e47efae41795ee590dc2ffb580f93f>}}

- Case 2: `this` with `bind()`  Method :

`bind()` method is the exact same as `call()` method but instead of immediately invoking the function, it will return a new function that we can invoke at a later time.

`bind()` provides two opportunities to call a function

{{< gist AkashRajvanshi d119d488ce87e9a6d510c436497dd995 >}}

- Case 3 : `this` with hard binding.

Hard Binding:  When binding is both explicit and strong.

our `showDetails.call(fullname)` is an explicit binding and when we put this statement in a function then it is called as strong binding and after the strong binding the value of `this` is not overridden.

even in window object its refer to the object `fullName`

{{< gist AkashRajvanshi 89ba426565b363e7996e5bafda065475 >}}

---
### 4. `new` binding :

new binding is nothing but we use 'new' keyword, so when we invoke a function with this 'new' keyword,  under the hood, the JavaScript interpreter will create a brand new object for us and call it `this`. Otherwise call as "Constructor Call".

{{< gist AkashRajvanshi 1b506a56978a8fbc2380b8e7263b3fc1 >}}

---
 ### 5. Lexical Binding

We use Arrow function to demonstrate lexical binding, as arrow function don't have its own `this` so Instead `this` is determined lexically.

lexical binding of an arrow function cannot be overridden.

{{< gist AkashRajvanshi 1f141d4ec786bb868610699ab0bd89a0 >}}

---
### 6. `window` Binding

Explain with an Example :

{{< gist AkashRajvanshi a0ee8f5dc08df47ed62052d6406365e9 >}}

___

## Summary

### Short overview of values that `this` will be working on:

1. `this` in constructor functions and `this` directly in a method point to the object itself.
2. `this` in a global context and `this` in a simple function point to `window`.
3. `this` in an arrow function always takes up the value of `this` in its surrounding scope.
4. `this` in an event listener points to the listening element.

### Short overview of binding rules that are used with 'this' :

1. Look where the function was invoked.
2. If there is an object on left side of dot operator, then `this` refer to this object (Implicit Binding)
3. If the function invoked with `call`, `apply`, or `bind`? then `this` explicitly refer to an object.(explicit Binding)
4. If the function invoked using the `new` keyword? then the `this` keyword is refer to a newly created object that was created by JavaScript interpreter. (`new` Binding)
5. If `this` inside of an arrow function? then it is refer lexically in the enclosing (parent) scope. (lexical Binding)
6. If we are in strict mode? then `this` keyword is `undefined`.
7. if we are in global scope or in simple function scope? then `this` keyword is refer to a `window` object.

___
