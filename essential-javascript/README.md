# Table of Contents

1. [Variable Declarations](#variable-declarations)
2. [`let` & `const`](#let-and-const) 
3. [Global Variables](#gloabal-variables)
4. [Coding and Naming Conventions](#coding-and-naming-conventions)
5. [Commas, Comments, Semicolons and Whitespace](#commas-comments-semicolons-and-whitespace)
6. [Strings](#strings)
7. [Implicit Coercion](#implicit-coercion)
8. [Iteration Statements](#iteration-statements)
9. [Block Scope vs Lexical Scope](#scope)
10.[Conditionals](#conditionals)

## Variable Declarations 

1.1 Correct way to declare variables is to prefix them with `var` 
```javascript
var foo = 1;
var bar = foo;
var quux = [];
var norf = {};
var baz = 'Paris';
```

1.2 Use const to declare your variables in ES6 
```javascript
const foo = 1;
```

1.3 Group const and let variable together for increased readability
```javascript
const bar = 2;
const list = getNames();
let i;
let attr = true;
```

1.4. Never do this, always declare with `var`
```javascript
foo = 100;
```

1.5. Undeclared variables throw errors in strict mode
```javascript
function foo() {
	var a = 1;
	z = 2; // Throws a ReferenceError in strict mode
}
```

1.6. Anti-patterns in ES5 and ES6
```javascript
const foo  = 10,
	    bar  = 15,
	    baz  = [],
	    norf = false;
```	

[**&#8593; Back to TOC**](#table-of-contents)     

## let and const

2.1 Favor const over let in ES6 if you don't need to reassign. const means that the identifier can't be reassigned; in other words using const creates an immutable binding.
```javascript
const baz = 75; // all of the below assignment operators will throw
baz = 24;
baz >>= 0b101010;
baz += 24;
baz /= 24;
```

2.2 The following will not throw, make a note of this pattern - clearly a const value can change
```javascript
const quux = {};
quux.norf = 75;
console.log(quux.norf); //=> 75
```

2.3 Use `let` when you need to reassign a variable.
```javascript
let norf = 'norf';
norf = 'quux';
console.log(norf); //=> `quux`
```

2.4 Additionally `let` is only visible in the for() loop */
```javascript
function testLet() {
	// x is not visible here
	for (let x = 0; x < 5; x++) {
		// x is visible here, and in the for() parens
	}
	// x is not visible out here
}
```

2.5 `let` works very much like `var`, the difference is that the scope of a `var`
variable is the entire enclosing funciton
```javascript
function testVar() {
	// x is visible here
	for (let x = 0; x < 5; x++) {
		// x is visible here too
	}
	// x is also visible here
}
```

2.6 All together now, we'll see more block scope patterns later on the function won't run, but it illustrates fundamental concepts
```javascript
function tricky() {
	{
		let a;
		{
			// ok since it's a block scoped name
			const a = "sneaky";
			// error, was just defined with `const` above
			a = "foo";
		}
		// ok since it was declared with `let`
		a = "bar";
		// error, already declared above in this block
		let a = "inner";
	}
}
```

[**&#8593; Back to TOC**](#table-of-contents) 

## Global Variables
    
> Note: It is impossible to avoid global JavaScript, something will always be dangling in the global scope. There are many approaches however, from simple to more complex, to not abuse global scope - after reading through this section you should already start understanding the power of patterns :/ We'll explore more patterns that minimize globals in classes, and modules

3.1 Always declare using an identifier your variables, if you don't it will clutter the global scope 
```javascript
const foo = Math.PI;
const bar = [];
var quux = new Wizardry();
let i;
delete foo; //=> returns `false`

// anti-pattern
foo = Math.PI;
bar = [];
window.x = 10;
delete foo; // returns `true`
```

3.2 It's very important to declare your variables, even in functions
```javascript
(function() {
  const norf = 'Laser Tag!';
  console.log(norf);  //=> `Laser Tag!`
})();

console.log(norf);  //=> undefined

// if you don't, they become global variables.
(function() {
  norf = 'Laser Tag!';
  console.log(norf);  //=> `Laser Tag!`
})();

console.log(norf);  //=> `Laser Tag` *whoops*

// in ES6, you can use an immediately invoked arrow function as well
(() => {
  const norf = 'Laser Tag!';
  console.log(norf);  //=> `Laser Tag!`
})();

console.log(norf);  //=> undefined
```

3.3 When global variables sneak into your code the can cause problems.
```javascript
var count = function() {
  for (i = 0; i < 10; i += 1) {
    console.log(i);
  }
};

count();  //=> 0 1 2 3 4 5 6 7 8 9

var countSilently = function() {
  for (i = 0; i < 10; i += 1) {
    // don't print anything;
  }
};

// both loops increment i at the same time, which causes strange behavior.
window.setTimeout(countSilently, 10);
window.setTimeout(count,         10);  //=> 2 3 7 8 9
```

3.4 You can use 'this' in method definitions to refer to attributes of the method's object.
```javascript
var obj = {
  name: 'quux',
  getName: function() {
    console.log(this.name);
  }
};

obj.getName();  //=> `quux`

// But 'this' does not follow the normal rules of scope in JavaScript.
var obj = {
  name: 'quux',
  getName: function() {
    window.setTimeout(function() {
      console.log(this.name);
    }, 3000);
  }
};

obj.getName(); //=> undefined *whoops*

// In fact, this got bound to the global object in the callback.
// To get around this, we use `that`, `self`, or whatever you decide

var obj = {
  name: 'quux',
  getName: function() {
    var that = this;
    window.setTimeout(function() {
      print(that.name);
    }, 3000);
  }
};

obj.getName();  //=> `quux`
```

3.5 The keyword `this` is dynamically assigned at the callsite of a function.When a 
function is invoked as a method, i.e. obj.method(), 'this' is bound to 'obj'. 
But when a function is invoked by itself 'this' is bound to the global object.
```javascript
var life = 'I love me some JavaScript!';

function meaning() {
  var life = 'Life in a function called meaning';
  console.log(this.life)
}

// This is true even of functions that were defined as a method.
var obj = {
  name: 'quux',
  getName: function() {
    console.log(this.name);
  }
};

// When the function is invoked without 'obj.' in front of it, 'this' becomes
// the global namespace.
var getName = obj.getName;
getName();  //=> undefined

// wanted to scratch the surface with the `this` keyword, tbd in later patterns
```

3.6 *Recap:* to avoid or help prevent the use of globals in your applications you can use the following patterns popular amongst JavaScript developers
```javascript
// a. generally you can use a closure in both ES5 & ES6
function fullName(firstName, lastName) {
  let intro = "Your name is ";
  // this inner function has access to the outer function's variables, including the parameter​
  function makeFullName () {
    // intro is available here
    return intro + firstName + " " + lastName;
  }
  return makeFullName ();
}

console.log(intro); //=> undefined

// b. use an IIFE or IIAF as described above in (ii.)

// c: use the module pattern
const myModule = function () {

  // private variable(s)
  const myPrivateVar = "I am private, and can be accessed only from within the closure.";

  // private method(s):
  const myPrivateMethod = function () {
    console.log("I am private, and can be accessed only from within the closure.");
  };

  return  {
    myPublicProperty: "I'm accessible as myModule.myPublicProperty",
    myPublicMethod: function () {
      console.log("I'm accessible as myModule.myPublicMethod.");

      // Within myModule, I can access "private" vars and methods:
      console.log(myPrivateVar);
      console.log(myPrivateMethod());

      // The native scope of myPublicMethod is myModule; we can
      // access public members using "this":
      console.log(this.myPublicProperty);
    }
  };
}();

// d: The easiest way however is to wrap your code in a closure and manually expose those
// variables you need globally

(function() {
  // Your code here
  const foo = 1;
  const baz = 2;
  let quux = 'OOP';

  // Expose to global
  window['foo'] = foo;
})();

console.log(foo); //=> 1 *voila!*
```    

[**&#8593; Back to TOC**](#table-of-contents)  
	    
## Coding and Naming Conventions

4.1 Be courteous to those who read your code and allow for context in your naming - be descriptive and try to avoid one letter names
```javascript
// be descriptive
function initialize() {
  // ...do stuff...
}

// anti-pattern, don't do this
function i() {
  // ...do stuff...
}
```

4.2 A few simple rules for naming things that you should try and adhere to
```javascript
/*
 * use upper and lower case letters (A - Z, a - z) and digits (0 - 9) to form names
 * do not use the underscore, _ or dollar sign, $, and avoid use of international characters
 * most variables and functions should start with a lower case letter
 * constructor functions and Classes should be capitalized
 * constructor functions that must be used with the `new` prefix should start with a capital letter
 */
const result = [];
const myErrorClass = document.getElementsByClassName('error');
const setupProxy = function proxy() {
  // ...do stuff...
};

// ES6 Class declaration
class Polygon {
  constructor(height, width) {
    this.name = 'Polygon';
    this.height = height;
    this.width = width;
  }
  get area() {
    return this.height * this.width;
  }
  set area(value) {
    this.area = value;
  }
}

let rhombus = new Polygon(100, 100);

// ES5 Constructor function
const Polygon = function Polygon(height, width) {
  this.name = 'Polygon';
  this.height = height;
  this.width = width;
};

Polygon.prototype.getArea = function() {
  return this.height * this.width;
};

Polygon.prototype.setArea = function() {
  this.area = value;
};

let parallelogram = new Polygon(400, 300);
```

4.3 Variables should be declared before they are used and should be descriptive preferably variables should be alphabetical order
```javascript
const age = 0;
const adult = true;
const gender = '';
const isTeenager = !!adult;
const location = {};
```

4.4 When to use camel vs pascal case
```javascript
// Use camelCase only when naming functions, objects, variables and instances
const myObj = {};
const formData = [];
function emptyArray() {
  // ...do stuff...
}
const fluffyWhale = new Mammal();

// anti-pattern, don't do this
const myobj ={};
let formdata-rules = [];
function empty_array() {
  // ...do stuff...
}
```

4.5 Array and Object Literals
```javascript
// always use the literal notation {}, [] instead of new Object() or new Array()
let myObj = {};
let myArr = [];
myArr.splice(0, 1);
```

4.6 Function declarations
```javascript
/* 1a. functions should be declared before they are used,
 * 2a. there should be no space between the name of a function and it's invocation left of
 * parens, but one space on the right of parens where indicated to start body {
 * 3a. The body of a function should be indented two spaces
*/
function changeName(oldName, newName) {
  const oldName = document.getElementById('name').value;
  function capitalizeName() {
    return newName = oldName.toUpperCase();
  }
  return capitalizeName()
}

// b. named function expressions read well too
const changeName = function name(oldName, newName) {
  const oldName = document.getElementById('name').value;
  function capitalizeName() {
    return newName = oldName.toUpperCase();
  }
  return capitalizeName()
};

// c. never use new (function constructor) to create a function
const foo = new Function('x', 'y', 'return x / y * 10');

// d. when passing an anonymous function, use the arrow function notation so it executes in the
// context of `this`
[100, 99, 98, 97, 96].filter((ele) => {
  return ele % 2 == 0;
});

// anti-pattern, don't do this
[100, 99, 98, 97, 96].filter(function(ele) {
  return ele % 2 == 0;
});
```

4.7 Statements
```javascript
/* 1a. compound statements should be indented with 2 spaces
 * 2a. the should be space between a keyword followed by parens () - denotes not an invocation
 * 3a. the left curly brace { should be at the end of the line that begins the compound statement
 * 4a. the right curly brace } should end the compound statement
 * 5a. the return value expression must start on the same line as the return keyword
 * 6a. semicolon should be used at teh end of a simple statement
 */
if (foo <= bar) {
  console.log('The message');
}

if (x > y) {
  return true;
} else {
    return false;
}

if (z % 2 == 0) {
  return false;
} else if (z % 2 == 1) {
    return true;
} else {
    return;
}

// b. for statement
for (let i = 0; i < 10; i++ ) {
  if (i < 5) {
    console.log(i);
  }
}

// c. switch statement
switch (constName) {
  case "John":
  case "Moe":
  case "Aisha":
    console.log('Hey');
    break;
  default:
    alert('Default case');
}

// d. while statement
var x = 0;
var y = 0;

while (x < 3) {
  x++;
  y += x;
}

// e. try, catch statement
try {
  if (x == "") {
    throw "is Empty";
  }
  if (isNaN(x)) {
    throw "not a number";
  }
  if (x > 10) {
    throw "too high";
  }
  if (x < 5) {
    throw "too low";
  }
}
catch(err) {
  console.log("Input " + err);
}
```

[**&#8593; Back to TOC**](#table-of-contents)  

## Commas, Comments, Semicolons and Whitespace

5.1 Use commas before as the last token before line breaks
```javascript
// a. for example, this is good practice
const x = {
  firstName: 'Moe',
  lastName: 'Khan',
  age: 25
};

const y = { firstName: 'Ahad', lastName: 'Bokhari', age: 25 };

// b. you can even leave a trailing comma which leads to cleaner git diffs
const z = {
  firstName: 'Moe',
  lastName: 'Khan',
  age: 25,
};

// anti-pattern, don't do this
const y = {
    firstName: 'Moe'
  , lastName: 'Khan'
  , age: 25
};
```

5.2 It's useful for those reading your code (and perhaps yourself) to see comments. Be clever with the usage of comments, don't clutter code or leave erroneous comments which make programs even harder to read

```javascript
// a. use /* ... */ for multiline comments
/**
 * Multi-line comments are good to use for
 * text that spans multiple lines
 */
const multiLine = `Text that spans
                   multiple lines`;

// anti-pattern, don't do this

// Multi-line comments are good to use for
// text that spans multiple lines
// @param {Number} id
// @return {Node} node

// b. use `//` for single line comments

// how three dots changed javascript
let cold = ['autumn', 'winter'];
let warm = ['spring', 'summer'];

//construct an array
[...cold, ...warm];

// anti-pattern, don't do this
let cold = ['autumn', 'winter']; // how three dots changed javascript
let warm = ['spring', 'summer'];
[...cold, ...warm]; // construct an array

// c. start comments with a space for readability

// @todo: find and refactor these variables to camelCase (fullName)
const fullname = '';

// anti-pattern, don't do this
//@todo: find and refactor these variables to camelCase
```

5.3 There is alot of debate around this topic, but you should use semicolons in your code. Ultimately, it's a team / organizational decision and consistency is key

```javascript
// a. required when two statements are on the same line
for (let i = 0; i < items.length; i++) {
  // ...do stuff...
}

// b. required when separating common statements
let i;
i = 5;
i = i + 1;
i++;
const x = 9;
const fun = function() { console.log(foo) };
console.log("hi");

// c. some places to avoid use of semicolons, usually after closing a `}` bracket

// no semicolons after };
if (condition) { console.log(x) } else { console.log(y) }
for (let value of iterable) { console.log(x) }
while (condition) { console.log(x) }

// d. there are exceptions of course

// although you shouldn't use the do statement, note the semicolon
do { (console.log(x)) } while (condition);

// function expression
const func = function() { console.log(foo) };

// object literal
const x = {
  firstName: 'Moe',
  lastName: 'Khan',
  age: 25
};

// e. use a semicolon to before an IIFE which helps separate when you cat and then minify two self invoking functions
;(function () {
  const magicPower = 'Ray of Light';
  return magicPower;
})();
```

5.4 Drastically improve readability by using consistent rules in your whitespace which proved significant benefit

``` javascript
/* 1a. Blank lines improve readability categorized logically related sections of code
 * 2a. Every `,` should be followed by a space or a line break
 * 3a. Each `;` semicolon at the end of a statement should be followed with a line break
 * 4a. Each `;` semicolon in the control part of a `for` statement should be followed with a space
 */

let iterable = [10, 20, 30];

for (let i = 0; i < iterable.length; i++) {
  console.log(i);
}

// b. indent with two spaces
function quuz() {
  const name = '';
}

// c. the word function is always followed with one space
function baz(x) {
  return x;
}

// d. always place one space before the leading brace
function mul(x, y) {
  return x * y;
}

// anti-pattern, don't do this
function mul(){
  return x * y;
}

// e. a blank space should not be used between a function value and it's invoking
// anti-pattern, don't do this
function norf (y) {
  return y;
}

// f. in control statements, place one space before the opening parens (`if`, `while`)
if (Number.MAX_SAFE_INTEGER) {
  // ...do stuff...
}

while (z < 10) {
  n++;
}

// g. @todo: readability and maintainability for lines of code longer than 120 characters

// h. don't add spaces inside your parens or brackets, but add spaces inside curly braces
const bar = { key: 'value' };
const foo = [...baz];
console.log(foo[0]);

// anti-pattern, don't do this
const baz = {key: 'value'};
const norf = [ ...baz ];
console.log( foo[0] );
console.log( foo[ 0 ] );

// i. use indentation for long method chains
var text = g.selectAll("text")
  .data(data);

text.attr("class", "update");
text.enter()
  .append("text")
    .attr("class", "enter")
    .attr("x", function(d, i) { return i * 32; })
    .attr("dy", ".35em")
    .merge(text)
  .text(function(d) {
    return d;
  });

// j. leave a blank line after blocks and before the next statement
const foo = function() {
  let arr = [];
  function baz() {
    console.log(arr + 1);
  }

  return baz();
};

// k. end files with a single newline character
function bar(baz) {
  return baz;
}↵  
```

[**&#8593; Back to TOC**](#table-of-contents) 

## Strings

6.1 Use single quotes for strings
```javascript
// a. this is good practice
const foo = 'I am a good string';

// b. anti-pattern, don't do this
const bar = "I am a cool string";

// c. anti-pattern, don't use template literal syntax to construct simple single line strings
const norf = `I am an awesome string`;
```

6.2 Best practices when dealing with longer strings
```javascript
// a. a string with less than 100 chars
const baz = 'This is a string with less than 100 characters and should be written on 1 line without breaks';

// b. a string with over 100 characters should not include new line breaks nor concatenation syntax
const str = 'This is a string with more than 100 characters and should be written on one line without breaks like so';


// c. anti-pattern, don't do the below
const badHabit = 'This is a string split by new lines \
                  which is a really bad habit to form, \
                  so never do this. Keep your code \
                  readable at all times.';

const badForm = 'This is a string split by new lines' +
                 'which is a really bad habit to form' +
                 'so never do this. Keep your code' +
                 'readable at all times.';
```                 

6.3 Use template literals for multi line strings
```javascript
// a. this is a good habit to form
const warningMsg = `If you choose this option you cannot
                    recover the items unless you have chosen
                    save to cache in your preferences page`;

// b. you'll see the use of template strings in frameworks like Angular 2.0
const myObj = {
  template:`
    <h1>{{title}}</h1>
    <h2>{{employee.name}} details!</h2>
    <div><label>id: </label>{{employee.id}}</div>
    <div><label>name: </label>{{employee.name}}</div>
    `
};
```

6.4 Use template literals for embedding expressions
```javascript
// a. this makes your code more readable
const a = 100;
const b = 15;
console.log(`The sum of 100 and 15 is equal to: ${a + b}`);
```

[**&#8593; Back to TOC**](#table-of-contents) 

## Implicit Coercion

7.1 JavaScript can be surprisingly forgiving when it comes to type errors, be wary of this in your applications 
```javascript
5 + true; // => 6 
```

7.2 In many cases rather than raising an error JavaScript coerces a value to the expected type by following automatic processes. When you mix numbers and strings, JavaScript breaks in favor of strings 
```javascript
5 + "5" // => 55
"5" + 5 // => 55
```

7.3 Avoid using '==' with mixed types, this helps the reader to keep clear there is no conversion involved in the comparison
```javascript
const today = new Date();
if (form.month.value === (today.getMonth() + 1) && // strict '==='
     form.day.value === today.getDate()) { // strict '==='
       //..do something
}
```

[**&#8593; Back to TOC**](#table-of-contents) 

## Iteration Statements

> The syntax of a for loop includes the following:
  a: initialization - typically used to initialize a counter variable
  b: condition - expression to be evaluated before each loop iteration
  c: final expression - expression to be evaluated at the end of each loop iteration
  d: statement: statement to execute as long as the condition is truthy

8.1 A `for loop` repeats until a specified condition evaluates to false. A contrived example
```javascript
for (let i = 0; i < 5; i++) {
  console.log(i);
    // statements
}

// => 0, 1, 2, 3, 4
```
8.2 A little more involved example, loop through inputs in form and find empty input
```javascript
const elements = document.getElementsByTagName("input");
for (let i = 0; i < elements.length; i++) {
  if(elements[i].value == "")
    alert('empty');
      // ..do something
}
```

8.3 The best way to loop through an array is using the `for loop` not `for-in` as we don't want to enumerate over props previously associated to the Object
```javascript
const myArray = ["xx", "zz"];
Object.prototype.newMethod = "cc";

// our humble for loop
for (var i=0; i < myArray.length; i++) {
  console.log(myArray[i]); // => "aa", "bb"
}
```

> A for-in statement loops through the props of an enumerable object
  The block of code inside the loop will be executed once for each property,
  but bear in mind that it will iterated inherited enumerable props via the
  property chain.

9.1 A simple construct for a for..in

```javascript
const obj = {
  name: 'Ahad',
  age: 22,
  occupation: 'developer'
};

for (let i in obj) {
  // ...use with object.hasOwnProperty
  if (obj.hasOwnProperty(prop)) {
    // ...do something
  }

}
```

9.2 One construct can be different from the other. For example:
```javascript
let a = []; // create a new empty array.
a[5] = 5;   // perfectly legal JavaScript that resizes the array.

for (let i = 0; i < a.length; i++) {
  // Iterate over numeric indexes from 0 to 5, as everyone expects.
  console.log(a[i]);
}

/* ==>:
 undefined
 undefined
 undefined
 undefined
 5
 */
```

9.3. Now using for in:
```javascript
var a = [];
a[5] = 5;
for (var x in a) {
  // shows only the explicitly set index of "5", and ignores 0-4
  console.log(x);
}

/* Will display:
 5
 */
```

9.4 Order is not preserved as well as inherited properties will also be enumerated
```javascript
Array.prototype.last = function () { return this[this.length-1]; };

for (var p in []) {
  // ...an empty array
  // ...last will be enumerated
}
```

> The for...of statement is a native method introduced in ES6 for iterating  * over iterable collections, or objects that have a   
 [Symbol.iterator] property.

9.5 The for..of uses the following syntax:
```javascript
for (variable of iterable) {
  //...do something
}
```

9.6 By default objects are not iterables, therefore the for..of doesn't work well in such cases. However for..of works perfectly with arrays and strings
```javascript
const array = ['a', 'b', 'c', 'd'];
for (let item of array) {
  console.log(item)
}
// ==> a, b, c, d

const str = 'Ahad Bokhari';
for (let char of str) {
  console.log(char)
}
// ==> A, h, a, d, ,B, o, k, h, a, r, i
```

/* `do while` */

/* while */

/* for Each */

## Scope

> Most programming languages use lexical scope, or scope based on variables defined at author time by the programmer.
 Lexical scoping defines how variable names are resolved in nested functions: inner functions contain the scope of parent 
 functions even if the parent function has returned.

10.1 Take a look at this block of code
```javascript
// function foo is the global scope
function foo(x) {
  // this encompasses the scope of foo
  var y = x * 5;

  function baz(z) {
    // encompasses the scope of bar
    console.log( x, y, z );
  }
  baz(y * 5);
}

foo(2);
// => 2 10 50
```

10.2 Scope based hiding techniques are powerful modules, do the scope of foo is nested, ultimately hiding what gets defined inside of it from the outside world
```javascript
var a = 1;
function foo() {
  var a = 10;
  console.log(a);
}
console.log(a);
// => 1

foo();
// => 10
```

10.3 It's important to note that we can artificially introduce scope by creating new functions and immediately invoking them.

> (ii). ES6 introduces block scoping and antything between { } introduces it's own
  scope. `let` statement declares a block scope local variable.

10.4 Consider the following code. Note that the { } not only creates new scope, it also invokes the console.log statement
```javascript
{
  let quux = "Hello World from ES6";
  console.log(quux);
  // => Hello World from ES6
}

{
  let foo = "This has a different scope";
  console.log(foo);
  // => This has a different scope
}

console.log(quux);
// => Reference Error: quux is not defined
```

10.5 Consider the difference between using `var` and `let` when thinking about scoping rules. The main difference is that the scope of var is the entire enclosing function
``` javascript
function testVar() {
  var x = 1;
  if (true) {
    var x = 2;  // same variable!
    console.log(x);  // => 2
  }
  console.log(x);  // => 2
}

function testLet() {
  let x = 1;
  if (true) {
    let x = 2;  // different variable
    console.log(x);  // => 2
  }
  console.log(x);  // => 1
}
```

10.6 Finally, some more block scoped `let` ...
```javascript
let outer = "outer";
{
  let inner = "inner";
  {
    let nested = "nested"
  }
  console.log(inner); // => you can access `inner`
  console.log(nested) // => throws error
}
// you can access `outer` here
// you cannot access `inner` and `nested` here
```javascript

## Conditionals

> (i.) Conditionals control behavior in JavaScript and determine whether or not pieces of code can run. They execute or skip other statements        * depending on the value of a specified expressions.

10.1 The most basic conditional is the if / else statement. The statement has two different forms, the first being:
```javascript
if (expression) {
  // if the expression is truthy the statement is executed
  ...statement
}
```

10.2 A single statement is required after the if keyword and parenthesized expression, however you can also use multiple statements within a block
```javascript
if (!user) {
  name = '';
  address = '';
  language = 'JavaScript';
}
```

10.3 The second form executes if the expression is false and introduces an else clause to defer the control to
```javascript
if (expression) {
  ...statement1
} else {
  ...statement2
}
```

10.4 Follow the example below
```javascript
if (n > 1) {
  console.log('You have 1 new notification');
} else {
  console.log('You have ' + n + ' new messages');
}
```

10.5 Beware nested if statements with else clauses, use cauton that the else clause goes where it is appropriately needed
```javascript
const i = 1;
const k = 2;

if (i == 1) {
  if (k === 2) {
    console.log('k equals 2');
    // a good rule is the else clause is part of the nearest if statement
  } else {
    console.log('k does not equal 2');
  }
}
```

> The else / if statement statement is used when you need to execute many pieces of code. There is no else if in JavaScript,
  but it is a frequently used programming idiom

10.6 Familiarize yourself with the structure
```javascript
if (expression) {
  // ...execute code block #1
}
else if (expression) {
  // ...execute code block #2
}
else if (expression) {
  // ...execute code block #3
}
else {
  // ...if all else fails, execute block #4
}
```

10.7 else ...if without the placeholders, in the real world
```javascript
if (quux === 1) {
  // ...execute code block #1
} else if (quux === 2) {
  // ...execute code block #2
} else if (quux === 3) {
  // ...execute code block #3
} else {
  // ...if all else fails, execute block #4
}
```

10.8. the above else ..if idiom is much preferable and readble to this
```javscript
if (quux === 1) {
// Execute code block #1
}
else {
  if (quux === 2) {
  // Execute code block #2
  }
  else {
    if (quux === 3) {
    // Execute code block #3
    }
    else {
    // If all else fails, execute block #4
    }
  }
}
```
> The switch statement simplifies both the appearance and the performance of multiple conditions. You can rewrite the previous example using a
  switch statement as follows:

10.9 Familiarize yourself with the structure
```javscript
switch(quux) {
  case 1: // Start here if foo == 1
  // Execute code block #1.
    break;
  // Stop here
  case 2: // Start here if foo == 2
  // Execute code block #2.
    break; // Stop here
  case 3: // Start here if foo == 3
  // Execute code block #3.
    break; // Stop here
  default: // If all else fails...
  // Execute code block #4.
    break; // stop here
}
```

10.10. A simple example
```javascript
function convert(x) {
  switch(typeof x) {
    case 'number': // Convert the number to a hexadecimal integer
      return x.toString(16);
    case 'string': // Return the string enclosed in quotes
      return '"' + x + '"';
    default: // Convert any other type in the usual way
      return String(x);
  }
}
```

[**&#8593; Back to TOC**](#table-of-contents) 