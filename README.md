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

When multiple rules apply, the rule higher on the list wins.
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
