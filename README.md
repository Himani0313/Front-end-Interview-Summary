# Front-end-Interview-Summary
Gist of most important javascript concepts


## THIS Keyword rules
1. If the new keyword is used when calling the function, this inside the function is a brand new object.
```
function ConstructorExample() {
    console.log(this);
    this.value = 10;
    console.log(this);
}
new ConstructorExample();
// -> {}
// -> { value: 10 }
```

2. If apply, call, or bind are used to call a function, this inside the function is the object that is passed in as the argument.
```
function fn() {
    console.log(this);
}
var obj = {
    value: 5
};
var boundFn = fn.bind(obj);
boundFn();     // -> { value: 5 }
fn.call(obj);  // -> { value: 5 }
fn.apply(obj); // -> { value: 5 }
```

3. If a function is called as a method — that is, if dot notation is used to invoke the function — this is the object that the function is a property of. In other words, when a dot is to the left of a function invocation, this is the object to the left of the dot. (ƒ symbolizes function in the code blocks)
```
var obj = {
    value: 5,
    printThis: function() {
        console.log(this);
    }
};
obj.printThis(); // -> { value: 5, printThis: ƒ }
```

4. If a function is invoked as a free function invocation, meaning it was invoked without any of the conditions present above, this is the global object. In a browser, it’s window
```
function fn() {
    console.log(this);
}
// If called in browser:
fn(); // -> Window {stop: ƒ, open: ƒ, alert: ƒ, ...}
```
*Note that this rule is the same as rule 3 — the difference is that a function that is not declared as a method automatically becomes a property of the global object, window. This is therefore an implicit method invocation. When we call fn(), it’s interpreted as window.fn(), so this is window.*

5. If multiple of the above rules apply, the rule that is higher wins and will set the this value.

6.  If the function is an ES2015 arrow function, it ignores all the rules above and receives the this value of its surrounding scope at **the time it’s created**. To determine this, go one line above the arrow function’s creation and see what the value of this is there. It will be the same in the arrow function.
```
const obj = {
    value: 'abc',
    createArrowFn: function() {
        return () => console.log(this);
    }
};
const arrowFn = obj.createArrowFn();
arrowFn(); // -> { value: 'abc', createArrowFn: ƒ }
```


obj.printThis() falls under rule 3 — invocation using dot notation. On the other hand, print() falls under rule 4 as a free function invocation. For print() we don’t use new, bind/call/apply, or dot notation when we invoke it, so we go to rule 4 and this is the global object, window.
```
var obj = {
    value: 'hi',
    printThis: function() {
        console.log(this);
    }
};
var print = obj.printThis;
obj.printThis(); // -> {value: "hi", printThis: ƒ}
print(); // -> Window {stop: ƒ, open: ƒ, alert: ƒ, ...}
```

**When multiple rules apply, the rule higher on the list wins.**
```
var obj1 = {
    value: 'hi',
    print: function() {
        console.log(this);
    },
};
var obj2 = { value: 17 };
```

If rules 2 and 3 both apply, rule 2 takes precedence.
```
obj1.print.call(obj2); // -> { value: 17 }
```

If rules 1 and 3 both apply, rule 1 takes precendence.
```
new obj1.print(); // -> {}
```


## When not to use arrow functions
1. Defining methods on an object

In JavaScript, the method is a function stored in a property of an object. When calling the method, this becomes the object that the method belongs to.

 1a.  Object literal
```
const calculate = {
  array: [1, 2, 3],
  sum: () => {
    console.log(this === window); // => true
    return this.array.reduce((result, item) => result + item);
  }
};
console.log(this === window); // => true
// Throws "TypeError: Cannot read property 'reduce' of undefined"
calculate.sum();
```
calculate.sum method is defined with an arrow function. But on invocation calculate.sum() throws a TypeError, because this.array is evaluated to undefined.
When invoking the method sum() on the calculate object, the context remains window. It happens because the arrow function binds the context lexically with the window object.
Executing this.array is equivalent to window.array, which is undefined.

The solution is to use a function expression or shorthand syntax for method definition (available in ECMAScript 6). In such a case this is determined by the invocation, but not by the enclosing context. Let’s see the fixed version:
```
const calculate = {  
  array: [1, 2, 3],
  sum() {
    console.log(this === calculate); // => true
    return this.array.reduce((result, item) => result + item);
  }
};
calculate.sum(); // => 6
```
Because sum is a regular function, this on invocation of calculate.sum() is the calculate object. this.array is the array reference, therefore the sum of elements is calculated correctly: 6.

 1b. Object Prototype

The same rule applies when defining methods on a prototype object.
Instead of using an arrow function for defining sayCatName method, which brings an incorrect context window:

```
function MyCat(name) {
  this.catName = name;
}
MyCat.prototype.sayCatName = () => {
  console.log(this === window); // => true
  return this.catName;
};
const cat = new MyCat('Mew');
cat.sayCatName(); // => undefined
```

use the old school function expression:
```
function MyCat(name) {
  this.catName = name;
}
MyCat.prototype.sayCatName = function() {
  console.log(this === cat); // => true
  return this.catName;
};
const cat = new MyCat('Mew');
cat.sayCatName(); // => 'Mew'
```

sayCatName regular function is changing the context to cat object when called as a method: cat.sayCatName().
    
2. Callback functions with dynamic context

this in JavaScript is a powerful feature. It allows changing the context depending on the way a function is called. Frequently the context is the target object on which invocation happens, making the code more natural. It says like “something is happening with this object”.
However, the arrow function binds the context statically on the declaration and is not possible to make it dynamic. It’s the other side of the medal in a situation when lexical this is not necessary.
Attaching event listeners to DOM elements is a common task in client-side programming. An event triggers the handler function with this as the target element. Handy usage of the dynamic context.

The following example is trying to use an arrow function for such a handler:
```
const button = document.getElementById('myButton');
button.addEventListener('click', () => {
  console.log(this === window); // => true
  this.innerHTML = 'Clicked button';
});
```
this is window in an arrow function that is defined in the global context. When a click event happens, browser tries to invoke the handler function with button context, but arrow function does not change its pre-defined context.
this.innerHTML is equivalent to window.innerHTML and has no sense.

You have to apply a function expression, which allows to change this depending on the target element:
```
const button = document.getElementById('myButton');
button.addEventListener('click', function() {
  console.log(this === button); // => true
  this.innerHTML = 'Clicked button';
});
```

3. Invoking constructors

this in a construction invocation is the newly created object. When executing new MyFunction(), the context of the constructor MyFunction is a new object: this instanceof MyFunction === true.

```
const Message = (text) => {
  this.text = text;
};
// Throws "TypeError: Message is not a constructor"
const helloMessage = new Message('Hello World!');
```
The above example is fixed using a function expression, which is the correct way (including the function declaration) to create constructors:
```
const Message = function(text) {
  this.text = text;
};
const helloMessage = new Message('Hello World!');
console.log(helloMessage.text); // => 'Hello World!'
```

4. Too short syntax

The arrow function has a nice property of omitting the arguments parenthesis (), block curly brackets {} and return if the function body has one statement. This helps in writing very short functions. Becomes difficult to understand in real world.

```
const multiply = (a, b) => b === undefined ? b => a * b : a * b;
const double = multiply(2);
double(3);      // => 6
multiply(2, 3); // => 6
```
multiply returns the multiplication result of two numbers or a closure tied with first parameter for later multiplication.
The function works nice and looks short. But it may be tough to understand what it does from the first look.

To make it more readable, it is possible to restore the optional curly braces and return statement from the arrow function or use a regular function:
```
function multiply(a, b) {
  if (b === undefined) {
    return function(b) {
      return a * b;
    }
  }
  return a * b;
}
const double = multiply(2);
double(3);      // => 6
multiply(2, 3); // => 6
```

## var let const

1. var
 1a. var variables can be re-declared and updated
 1b. var variables are hoisted to the top of its scope and initialized with a value of undefined
 
problems with var - 
```
var greeter = "hey hi";
var times = 4;

if (times > 3) {
    var greeter = "say Hello instead"; 
}

console.log(greeter) //"say Hello instead"
```
So, since times > 3 returns true, greeter is redefined to "say Hello instead". While this is not a problem if you knowingly want greeter to be redefined, it becomes a problem when you do not realize that a variable greeter has already been defined before. 
If you have use greeter in other parts of your code, you might be surprised at the output you might get. This might cause a lot of bugs in your code. This is why the let and const is necessary.

2. let
 2a. let is block scoped
 2b. let can be updated but not re-declared.
 2c. Hoisting of let
Just like var, let declarations are hoisted to the top. Unlike var which is initialized as undefined, the let keyword is not initialized. So if you try to use a let variable before declaration, you'll get a Reference Error

3. Const
 3a. const declarations are block scoped
 3b. const cannot be updated or re-declared (values can be updated in object, not directly for the const)
 3c. Just like let, const declarations are hoisted to the top but are not initialized.
 
So just in case, you missed the differences, here they are :

 * var declarations are globally scoped or function scoped while let and const are block scoped.

 * var variables can be updated and re-declared within its scope; let variables can be updated but not re-declared; const variables can neither be updated nor re-declared.

 * They are all hoisted to the top of their scope but while varvariables are initialized with undefined, let and const variables are not initialized.

 * While var and let can be declared without being initialized, const must be initialized during declaration.
 

## variable hoisting

JavaScript Engine does its job in two phases: the memory creation phase and the execution phase, and that our code won’t be executed until the second phase.

```
console.log(x)

// ReferenceError: x is not defined
```

```
console.log(x)
var x

// undefined
```
The key here is that x is defined and available before its declaration — yes, this is a legitimate example of hoisting. Hence, the example 2 is practically same as:
```
var x
console.log(x)
```

Value of x to undefined is set by JavaScript engine. During the memory creation phase, it recognises variable declarations as it reads the code, initialises them to undefined and puts them into memory to be used during the execution phase.

**initialisations are not hoisted**
```
console.log(x)
var x = 10

// undefined
```
During the memory creation phase, the JavaScript engine recognised the declaration of x (var x), automatically initialised x to undefined, and made it available. However, as the initialisation (= 10) didn’t get hoisted, value of x stayed as undefined when the execution reached console.log at line 1.

## Function Hoisting
```
sayHello()

function sayHello () {
  function hello () {
    console.log('Hello!')
  }
  
  hello()
  
  function hello () {
    console.log('Hey!')
  }
}
```
Returns ----> 'Hey!'

```
sayHello()

function sayHello () {
  function hello () {
    console.log('Hello!')
  }
  
  hello()
  
  var hello = function () {
    console.log('Hey!')
  }
}
```
Returns ----> 'Hello!'

```
sayHello()

var sayHello = function () {
  function hello () {
    console.log('Hello!')
  }
  
  hello()
  
  function hello () {
    console.log('Hey!')
  }
}
```
Returns ---> TypeError

*Function Declaration vs Function Expression*
There are two ways to define a function with the function keyword in JavaScript — function declaration and function expression.
A function declaration starts with the function keyword, followed by the name of the function (sayHello), then a block of code to be executed when the function is called ({ console.log('Hello!') }).

On the other hand, a function expression allows you to define a function without a name and as part of non-functional code blocks. A typical usage of a function expression is to assign a function to a variable. Below, I’m defining an anonymous function, that is, function without a name, (function () { console.log(Hello!) }) and assigning it to a variable (var sayHello =), so I can refer to the function via sayHello later on.
```
var sayHello = function() {
  console.log('Hello!')
}
```

**Explanation**
During the memory creation phase, the JavaScript engine recognised a function declaration by the function keyword and hoisted it — in other words, the JavaScript engine made the function available by putting it into the memory, before moving on. That’s why could access the sayHello function prior to its declaration in the execution phase.
```
sayHello()

function sayHello () {
  console.log('Hello!')
}

// Hello!
```

*Function Expressions are Not Hoisted*
```
sayHello()

var sayHello = function () {
  console.log('Hello!')
}

// TypeError
```

During the memory creation phase, the JavaScript engine encounters the var keyword at line 3, at which point it expects a variable declaration to follow. JavaScript engine when encounters a variable declaration it hoists the variable with a value: undefined. And it doesn’t hoist the variable initiation.
The variable declaration (var sayHello) was hoisted with a value undefined. However, the variable initialisation (= function () { console.log(Hello!) }) wasn’t hoisted. Therefore, when the execution reached line 1 and tried to call sayHello, it failed, because undefined is not a function! Only after the sayHello variable is assigned to a function expression during the execution at line 3, can we call the function by sayHello(). 
