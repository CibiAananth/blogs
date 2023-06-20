---
title: "Hoisting in JavaScript: One last time"
seoTitle: "JavaScript Hoisting Unveiled: Deep Dive into JavaScript Internals"
seoDescription: "Dive into JavaScript Hoisting! Explore variable/function hoisting, understand 'let', 'const', 'var', and the Temporal Dead Zone in a fun way!"
datePublished: Tue Jun 13 2023 14:49:03 GMT+0000 (Coordinated Universal Time)
cuid: cliuefcbo000209l49x619icd
slug: js-hoisting
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1687269696237/7d0e1dca-fa62-4e24-95b9-91c8793ed215.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1686667560032/c48f7769-9242-4cf4-abac-84d59fddd13b.png
tags: javascript, web-development, webdev, reactjs

---

---

Hello JavaScript enthusiasts! Today, we're diving into one of JavaScript's most interesting behaviors: **Hoisting**. This strange-sounding term might be a source of confusion, but worry not! We're here to unravel its mysteries, all while keeping things fun and engaging. So grab your diving gear, it's time for a deep dive!

---

## **Hoisting - What's That?**

In the magical world of JavaScript, hoisting is a peculiar behavior where variable and function declarations are moved to the top of their containing scope during the compilation phase. But, let's make this clear: while it's helpful to picture declarations physically "moving up", what happens is JavaScript engine stores these declarations in memory during the parsing phase.

Now that we've briefly introduced hoisting, let's dive deeper into how different variable declarations are hoisted!

---

### **The Tale of 'var' Hoisting**

When a variable is declared using `var`, JavaScript hoists it to the top of its function or global scope. However, it's initialized with `undefined` until it encounters the assignment in the code.

```javascript
console.log(myVar); // Outputs: undefined
var myVar = 5;
console.log(myVar); // Outputs: 5
```

Here, `myVar` is known from the start, but it begins its life as `undefined`. Only when the assignment is reached, `myVar` finally becomes `5`.

Yet another example,

```javascript
var myVar = 5;
function print() {
  console.log(myVar); // undefined
  var myVar = 10;
}
print();
```

In the above code snippet. even though the value of `myVar` is initialized to `5` before console statement it still prints `undefined`. Why? let's apply the rules of variable hoisting again,

```javascript
// Equivalent JS hoisting under the hood
var myVar = 5;
function print() {
  var myVar;
  console.log(myVar); // undefined
  myVar = 10;
}
print();
```

Since `myVar` is redeclared again inside print, it is hoisted to `print` function scope and thus the value is `undefined`

---

### **'let' and 'const' Hoisting**

Unlike `var`, `let` and `const` are hoisted too, but they aren't initialized. Attempting to access them before declaration leads us to an error-ridden place, known as the Temporal Dead Zone (TDZ).

```javascript
console.log(myLet); // ReferenceError: myLet is not defined
let myLet = 5;
```

So, even though `myLet` is hoisted, and trying to access it in its TDZ throws a ReferenceError. Always declare `let` and `const` before use!

---

### **Temporal Dead Zone (TDZ) - An Uncharted Territory**

The TDZ is the period between entering scope and being declared where variables can't be accessed. It's JavaScript's way of encouraging us to declare variables before use.

```javascript
{
  // TDZ for myLet starts
  console.log(myLet); // ReferenceError: myLet is not defined
  let myLet = 5; 
  // TDZ for myLet ends
}
```

---

### **Function Hoisting - A Different Beast**

When it comes to function declarations, JavaScript treats them with special care. Both the name and body of function declarations are hoisted, allowing us to call the function even before it's declared!

```javascript
console.log(sum(1,2)); // Outputs: 3
function sum(a, b) {
  return a + b;
}
```

That's right! We can use `sum` even before its declaration, thanks to hoisting.

---

### **Arrow Functions - The Cool Kid**

Arrow functions, like function expressions, follow the hoisting rules of their variable declaration. If stored in a `var` variable, it's hoisted but initialized with `undefined`. If in a `let` or `const`, it resides in the TDZ until declaration.

```javascript
console.log(sum(1,2)); // ReferenceError: sum is not defined
let sum = (a, b) => a + b;
```

```javascript
console.log(sum(1,2)); // TypeError: sum is not a function
var sum = (a, b) => a + b;
```

Why does var implementation throw TypeError rather than ReferenceError?

According to `var`'s hoisting rules with the following representation,

```javascript
// Equivalent JS hoisting under the hood
var sum; // undefined
console.log(sum(1,2)); // TypeError: sum is not a function
sum = (a, b) => a + b;
```

Here's the value of `sum` is `undefined` during `sum(1,2)` but we're trying to invoke a function call which is a TypeError

---

To wrap up, understanding hoisting and its nuances allows you to read and write JavaScript more effectively. It might seem like a quirk, but it's an integral part of the JavaScript landscape. So the next time you stumble upon a tricky piece of JavaScript code, remember, hoisting might be at play!

Happy Coding!

---