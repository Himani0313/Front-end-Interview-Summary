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
