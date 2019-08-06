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

## SCOPE

1. Global Scope
The global scope is the outermost scope and is pre-defined even before you write a single line of code. (Window)
Global variables can be accessed and modified from any other scope.
```
// Global scope

var greet = 'Hello!' // Globally scoped

function changeGreet () {
  console.log('2: ', greet) // Accessible
  greet = 'Hey!' // Modified
  console.log('3: ', greet) // Accessible
}

console.log('1: ', greet) // Accessible
changeGreet()
console.log('4: ', greet) // Accessible

// 1: Hello! 
// 2: Hello!
// 3: Hey!
// 4: Hey!
```

2. Local Scope
Local scope is any scope created within the global scope. Every time a new function is declared, a new local scope gets created, and variables declared inside the function belong to that unique scope.
During the execution phase, local variables can only be accessed and modified within the same scope. As soon as the JavaScript engine finishes executing a function, it exits the local scope and moves back to the global scope, losing access to the variables within that local scope.
```
// Global scope

function sayHi () {
  // Local scope
  
  var greet = 'Hello!' // Localy scoped
  console.log('1: ', greet) // Accessible within the same scope
  
  greet = 'Hey!' // Modified within the same scope
  console.log('2: ', greet) // Accessible within the same scope
}

sayHi()
console.log('3: ', greet) // NOT accessible from outside the scope (global scope)

// 1: Hello!
// 2: Hey!
// ReferenceError: greet is not defined
```

can have multiple local scopes within the global scope. Each local scope is an isolated entity, so variables that belong to a scope is confined to that specific scope.

**Hoisting and Scope**
```
var greet = 'Hello!'

function sayHi () {
  console.log('2: ', greet)
  var greet = 'Ciao!'
  console.log('3: ', greet)
}

console.log('1: ', greet)
sayHi()
console.log('4: ', greet)


////////
1: Hello!
2: undefined
3: Ciao!
4: Hello!
```

## Execution Context ≠ Scope
It’s easier to demonstrate this with an example rather than with a wall of text. At this point, we just need to keep in mind that the following events happen, as the JavaScript engine starts to read your code.
The global execution context is created before any code is executed.
Whenever a function is executed (or called/invoked, these are all synonyms), a new execution context gets created.
Every execution context provides this keyword, which points to an object to which the current code that’s being executed belongs.

```
var globalThis = this

function myFunc () {  
  console.log('globalThis: ', globalThis)
  console.log('this inside: ', this)
  console.log(globalThis === this)
}

myFunc()

// globalThis: Window {...}
// this inside: Window {...}
// true
```
**_In JavaScript, execution context is an abstract concept that holds information about the environment within which the current code is being executed._**

the JavaScript engine creates the global execution context before it starts to execute any code. From that point on, a new execution context gets created every time a function is executed, as the engine parses through your code. In fact, the global execution context is nothing special. It’s just like any other execution context, except that it gets created by default.

```
var name = 'John'

function greet (name) {  
  return (function () {
    console.log('Hello ' + name)
  })
}

var sayHello = greet(name)

name = 'Sam'

sayHello()

\\ Hello John
```

## Closures

A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time.
To use a closure, define a function inside another function and expose it. To expose a function, return it or pass it to another function.
The inner function will have access to the variables in the outer function scope, even after the outer function has returned.

**Explaination**
* When code is run in JavaScript, the environment in which it is executed is very important, and is evaluated as 1 of the following:
 * Global code — The default environment where your code is executed for the first time.
 * Function code — Whenever the flow of execution enters a function body.
 * (…)
 * (…), let’s think of the term execution context as the environment / scope the current code is being evaluated in.

In other words, as we start the program, we start in the global execution context. Some variables are declared within the global execution context. We call these global variables. When the program calls a function, what happens? A few steps:
* JavaScript creates a new execution context, a local execution context
* That local execution context will have its own set of variables, these variables will be local to that execution context.
* The new execution context is thrown onto the execution stack. Think of the execution stack as a mechanism to keep track of where the program is in its execution

When does the function end? When it encounters a return statement or it encounters a closing bracket }. When a function ends, the following happens:
* The local execution contexts pops off the execution stack
* The functions sends the return value back to the calling context. The calling context is the execution context that called this function, it could be the global execution context or another local execution context. It is up to the calling execution context to deal with the return value at that point. The returned value could be an object, an array, a function, a boolean, anything really. If the function has no return statement, undefined is returned.
* The local execution context is destroyed. This is important. Destroyed. All the variables that were declared within the local execution context are erased. They are no longer available. That’s why they’re called local variables.


```
let multiply = function(x) {
    return function(y) {
        console.log(x*y);
    }
}

multiplyByTwo = multiply(2);
multiplyByTwo(3);

multiplyByThree = multiply(3);
multiplyByThree(3);
```


## Call, Apply, Bind

You can use call()/apply() to invoke the function immediately. bind() returns a bound function that, when executed later, will have the correct context ("this") for calling the original function. So bind() can be used when the function needs to be called later in certain events when it's useful.

1. Call
```
var obj = {name:"Niladri"};

var greeting = function(a,b,c){
    return "welcome "+this.name+" to "+a+" "+b+" in "+c;
};

console.log(greeting.call(obj,"Newtown","KOLKATA","WB"));
// returns output as welcome Niladri to Newtown KOLKATA in WB
```
The first parameter in call() method sets the "this" value, which is the object, on which the function is invoked upon. In this case, it's the "obj" object above.

The rest of the parameters are the arguments to the actual function.

**_Object to Array_**
```
[].slice.call(arguments)
```
**_Call Base class constructor from Child Class_**
```
let mammal = function(legs) {
    this.legs = legs;
}

let cat = function(legs, isDomesticated) {
    mammal.call(this,legs);
    this.isDomesticated = isDomesticated;
}
let lion = new cat(4,false);
```

2. Apply
```
var obj = {name:"Niladri"};

var greeting = function(a,b,c){
    return "welcome "+this.name+" to "+a+" "+b+" in "+c;
};

// array of arguments to the actual function
var args = ["Newtown","KOLKATA","WB"];  
console.log("Output using .apply() below ")
console.log(greeting.apply(obj,args));

/* The output will be 
  Output using .apply() below
 welcome Niladri to Newtown KOLKATA in WB */
```

Similarly to call() method the first parameter in apply() method sets the "this" value which is the object upon which the function is invoked. In this case it's the "obj" object above. The only difference of apply() with the call() method is that the second parameter of the apply() method accepts the arguments to the actual function as an array.

**_Apply functions to arrays_**
```
let numArray= [1,2,3];
Math.min(1,2,3) // Allowed but not feasible for arrays
Math.min.apply(null, numArray);
```

3. Bind
```
var obj = {name:"Niladri"};

var greeting = function(a,b,c){
    return "welcome "+this.name+" to "+a+" "+b+" in "+c;
};

//creates a bound function that has same body and parameters 
var bound = greeting.bind(obj); 


console.dir(bound); ///returns a function

console.log("Output using .bind() below ");

console.log(bound("Newtown","KOLKATA","WB")); //call the bound function

/* the output will be 
Output using .bind() below
welcome Niladri to Newtown KOLKATA in WB */
```

In the above code sample for bind() we are returning a bound function with the context which will be invoked later. 

The first parameter to the bind() method sets the value of "this" in the target function when the bound function is called. Please note that the value for first parameter is ignored if the bound function is constructed using the "new" operator.
The rest of the parameters following the first parameter in bind() method are passed as arguments which are prepended to the arguments provided to the bound function when invoking the target function.

```
let myObj = {
    asyncGet(cb) {
        cb();
    }
    parse() {
        console.log('parse called');
    }
    render() {
        this.asyncGet(function() {
            this.parse();
        });
    }
}

myObj.render() - Error as this.parse() not a function as this has scope of render
```

Solution::
```
render(){
    that = this;
    this.asyncGet(function() {
        that.parse(;
    }
}
```
OR
```
render() {
    this.asyncGet(function(){
        this.parse();
     }.bind(this));
}
```
