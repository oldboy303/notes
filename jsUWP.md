JS UNDERSTANDING THE WEIRD PARTS

## Execution Contexts and Lexical Environments

**Syntax Parsers**  
A program that reads your code, determines what it does and if its grammar is valid. (Compilers and/or Interpreters)

**Lexical Environment**  
*Where* something sits inside your code. This is important in programming languages in which where you write something determines its relation to other parts of your code - e.g. Javascript.

**Execution Context**   
A wrapper to help manage the code that is running. There might be many *lexical* environments in a given block of code, but which one is currently running is managed by execution contexts. It can contain things (e.g., variables, objects, functions, etc) beyond what you have written in your code. 

**Name / Value Pair** A name which maps to a unique value. A name may be defined more than once, but can only have one value in any given context. That value may be more name/value pairs.

**Objects**  
An object is just a collection of name/value pairs.

**The Global Environment and the Global Object**  
The *global context* is the base execution context in which all other code runs. This context creates a *global object* for you and a special variable called 'this' (these are created by the JS engine). In the browser at the global level, 'this' will refer to the window object (i.e., the global object for a browser); if the code is being run in some other environment (e.g., NodeJS), the global object will be something else and 'this' will refer to that object. 'Global' just means code not inside a function. (Note: inside of a function 'this' may or may not refer to the global object.)

An **Execution Context** in JS is always going to provide you with (at least) these three things:
* an object to which this refers
* the special variable 'this' itself
* a reference to the *outer environment* of the context if there is one (*null* in the case where you're already in the global context)

**The Execution Context and Hoisting**  
Basically. during the first ('creation') phase, the execution context is created and the JS engine creates the *global object*, the *this* variable and the reference to the outer environment (if there is one). Also during this phase, the JS engine will assign to a specific location in memory *functions in their entirety* and variable *names set to an initial value of undefined*. This is what gives rise to the phenomena of **_hoisting_**. 

Once this creation/first phase is complete, the JS engine will then *execute* your code line by line. 

Bear in mind that the engine does not *actually* move any variables or rearrange our code in any way before execution. Rather, by virtue of the fact that memory assignments have already taken place, it only *appears* that way and consequently we can utilize functions declared later in our code; for the same reasons, variables are likewise accessible, but will have the default value of 'undefined' until we reach the place in our code where they are assigned a certain value.

**Note** one should *never* set a variable equal to 'undefined' and should use 'null' instead when some placeholder value is required. This makes debugging much easier: *'Uncaught reference error: x is not defined'* errors are those where x is never defined in your code, whereas an 'undefined' error is one where we call a variable before we define it in our code, and for instances where we never assign a value to a declared variable, we will get a 'null' value. “Undefined” is actually a reserved keyword in JS. Note also that relying on hoisting can lead to problems in debugging unless one is very clear as to how this process plays out. 

**Single-threaded**  
one command is executed at a time... under the hood of your browser or node much more may be going on, but the behavior is *as if* one command is being executed at a time.

**Synchronous**  
one line of code executed at a time in order.

**Function Invocation and the Execution Stack**  
Whenever a function is *invoked ()*, a new execution context for that function is created - creation and hoisting - and is added to the 'top' of the *execution stack*. Whatever context is on the top of the stack must complete running its code before it is 'popped' off the stack and the parent context's code can continue running - this is a result of JS being single-threaded/synchronous in nature.

where the variables 'live' and how they relate to each other in memory. Every execution context has its own variable environment.

## The Scope Chain

This is where the *Lexical Environment* becomes important. When we call a variable, if it is not found in the current execution context, JS will look to the outer reference contained therein and search for the variable there. JS will continue this process until either the variable is found or the outer reference is null (i.e., it is in the global execution context and still can't find the variable) and issue the 'uncaught reference error...'. *It is the lexical environment that determines where that outer reference points*. For example:

```javascript
function b() {
  console.log(myVar);
}

function a() {
   myVar = 2;
   b();
}

var myVar = 1;
a();
```

Outputs '1'. This is because function b's lexical environment is the global execution context in this instance.

```javascript
function a () {

   function b() {
     console.log(myVar);
   }

   myVar = 2;
   b();
}

var myVar = 1;
a();
```

Outputs '2' because function b's lexical environment is now that of function a's execution context. (note that function b is no longer available in the global context)

The manner in which JS follows the series of outer references while searching for a variable is the *scope chain*.

## Scope, ES6 and 'let'  

**Scope**  
Where a variable is available in your code and if it's truly the same variable or a copy.

**let**
Introduces *block scoping* to JS. The variable is processed just as any other (e.g., hoisted during the creation phase of execution context creation), but is only available during the execution phase and only once the line on which it appears and only within the block ('{}') in which it occurs. This is true for both 'if' blocks and even 'for' loops - a new instance of the let variable for each iteration of the loop.

## Asynchronous Callbacks  

**Asynchronous**  
More than one at a time.

JS is of course synchronous. However, there are numerous other 'engines' at play in the browser (or node, for instance). In addition to the *execution stack* there is also an *event queue* in which events for which we are *listening* for are placed once the event occurs. The JS engine will then check the event queue whenever the execution stack is empty and periodically - this is know as the *event loop*. If it finds any events in the queue, it will then process any *event handlers* associated with the event. Hence while it might appear that JS can process events *asynchronously*, it is only the illusion of asynchronicity created by what is happening outside of JS - e.g., click events, http requests returning data, etc.,. **Note** that long-running functions can have a severe impact on performance as they will prevent the JS engine from checking the event queue until they are popped of the execution stack. 

## Types in JS 

**Dynamic Typing**  
The JS engine determines what type of data a variable holds, rather than us specifying it. Variables can hold different types of data because it is all figured out by the engine during execution. (Note that this is different than *static typed* languages like cC#, Java, etc.,) This is n=both extremely powerful and often the cause of difficulties and bugs.

**Primitive Types in JS**  
A primitive type is a data type that represents a single value - i.e., not objects. The six primitives in JS are: **_undefined, null, boolean, number, string, symbol (ES6)_**... **_NaN_** can also be considered a primitive.

**Operators**  
A special function that is syntactically (written) different. They take a given number of parameters (often two) and return a result. Checkout *infix, prefix* and *postfix notation*.

**Operator Precedence**  
which operator function gets called first. Functions are called in order of precedence and higher precedence goes first. Think order of operations in math.

```javascript
var a = 3 + 4 * 5;
console.log(a);
// a = 23 (multiplication comes first, higher precedence)
```

**Operator Associativity**  
The order in which operator functions get called - *left-to-right* or *right-to-left associativity* - when operators have the *same* precedence.

```javascript
var a = 2, b = 3, c = 4;
a = b = c;
console.log(a, b, c);
// a = 4 (assignment has right-to-left associativity)
```

**Operator Precedence Reference**  
[MDN Operator Precedence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence)

**Type Coercion**  
Converting a value from one type to another. This happens often in JS by virtue of it being dynamically typed. For example: 

```javascript
var a = 1 + '2' 
```

would set a equal to a string '12'. This can lead to any number of weird bugs...

```javascript
console.log(1 < 2 < 3); // Outputs 'true'
console.log(3 < 2 < 1); // Outputs 'true' as well
```

**explanation** comparison operators have left-to-right associativity and return a boolean. '3 < 2' returns 'false' which gets coerced to 0 and since, 0 < 1, returns 'true' on the second comparison.

**Note** because of type coercion and the often unexpected ways which it often works, it is almost always better to use **_strict equality and inequality ('===' or '!==')_** rather than '==' or '!='. The stricter equality comparisons check for *type equality* of two values without trying to coerce either. Proceeding thus can help avoid some surprising and difficult bugs which can arise because of type coercion, especially with regards to comparison operators. 

**Equality Comparisons and Sameness Reference**  
[MDN Comparisons and Sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)

(*Checkout* **Obect.is()**)

**Booleans and Existence**  
One instance where coercion can be helpful is in conditionals where we might have a value or maybe not. In instances where a value is *undefined, null, 0, or an empty string* the value will coerce to *false*, and in all(?) other instances will coerce to *true*. In other words, if a value exists and is not 0, the conditional will be triggered. Hence we often see code like this:

```javascript
var a;
if (a || a === 0) {
  // do something
}
```

**Default Values**  
Note that with coercion and the special case of the '||' operator we can setup default values for our parameters. Remember: operators are just functions. || returns the *first* truthy value of its parameters. Consequently, we can (and often see) default values set up like so:

```javascript
function greet (name) {
  name = name || '<Your name here>';
  console.log('Hello ' + name);
}
```

**Note** the ES6 syntax that accomplishes the same thing
**Note** checkout how libraries and frameworks use something similar to prevent global namespace collisions.

## Objects and Functions  

Objects a collections of name value pairs. The values can be either **_properties_** (primitives or other objects) or **_methods_** (functions attached to objects). 

**Note** *dot notation* (Object.property) is to be preferred over *bracket notation* (Object["property"]) unless one needs to access or create object properties *dynamically*. Also, *_object literal syntax_* is to preferred over '**new Object()**' for object creation - object literals allow us to initialize *and* create it on the same line.

**Namespace**  
a container for variables and functions, typically to keep variables or functions with the same name separate. In JS we don't have namespaces, but we can fake it using objects:

```javascript
// namespace collision
var greet = 'Hello';
var greet = 'Hola';
console.log(greet);

// fake namespaced
var english = {greet: 'Hello'};
var spanish = {greet: 'Hola'};
console.log(english.greet);
console.log(english.greet);
```

### Functions are Objects

**First class functions**  
Everything you can do with other types, you can do with functions - assign them to variables, pass them around, create them on the fly, etc.,.

Functions are also like other objects, though they are a somewhat special case. One can attach primitives to functions, other objects, other functions, etc.,. The name function of a function is optional - we can have *anonymous* functions. All functions have a *code* property - the actual code that a function runs when it is invoked. In other words, the code is not the function itself, the function is an object with the code attached (the code property of the function). The code property of a is then invocable with '**()**'.

### Function Statements and Expressions

**Expression vs Statement**  
An expression is a unit of code that results in a value - it doesn't have to save to a variable. A statement doesn't result in a value - i.e., it just does work.

**Function Expression**:  

```javascript
var greet = function() {
  console.log('Hi');
}
```

The code above returns a value, namely, the function object.

**Function Statement**  

```javascript
function greet() {
  console.log('Hi');
}
```

This above is placed into memory (during hoisting), but does not return a value outright when the code is run.

**By Value vs By Reference**  
Primitives are passed *by value*, which simply means that a copy of the value is stored in a different location in memory. So...

```javascript
var a = 3, b = a;
console.log(a);
console.log(b);

a = 2;
console.log(a);
console.log(b);
// Outputs 2 and 3
```

In this case, or indeed any case involving primitive values, the two variables each point to a different location in memory, hence, when we change a's value on line after assigning b equal to a, b's value remains unchanged. 

Objects and functions, however, are *passed by reference* and point to the same location in memory. So...

```javascript
var a = {name: 'Joe'};
var b = a;
console.log(a);
console.log(b);

a.name = 'Mary';
console.log(b);
```

As these two variables point to the same location, any change in one will be reflected in the other.

**Note** Importantly, this can even happen when an object/function is passed as an argument to a function and that function mutates some property/method!

**Objects, Functions and 'this'**  
When we create a function either as an expression or a statement, the 'this' keyword points to the global object. However, when we are dealing with a method - a function attached to an object - the 'this' keyword will point to the object that contains the method. Note, however, that if we create another function inside a method, the 'this' keyword will point to the global object... this is reasonably considered a bug in JS. This bug is what gives rise to the '**var self = this**' type of coding device that we see so often employed in JS and which binds the reference to the object in which it appears. 

**'arguments'** is a keyword just like 'this' that is available to us in functions and which contains an array-like list of all the arguments past to a function.
(**checkout spread in ES6**)

**Function Overloading**  
This is a concept which exists in some other languages, but isn't really necessary in JS because of first class functions. Basically, it is the idea that one can have different functions with the same name but with different parameters and which vary in their output based on that - obviously this wouldn't work in JS as functions are objects and thus couldn't share a name. Yet, while this isn't possible, we can mimic it in a way...

```javascript
function greet(firstname, lastname, language) {
    if (language === 'en') {
      ('Hello ' + first name + last name);
    }
    if (language === 'es') {
      console.log('Hola ' + first name + last name);
    }
}

function greetEn(firstname, lastname) {
  greet(firstname, lastname, 'en');
}

function greetEs(firstname, lastname) {
   greet(firstname, lastname, 'es');
}

greetEn('John', 'Doe');
greetEs('John', 'Doe');
```

**Syntax Parsers**  
Remember that your code isn't run directly, but is interpreted by the syntax parser and can even make changes to your code. 

**Automatic Semicolon Insertion**  
DANGEROUS Semicolon use is optional in core JS. This is because the syntax parser interprets a carriage return character (invisible) as a semicolon where it would be necessary to make your code run rather than throw an error (e.g., after it sees the keyword 'return' and a carriage return). This can lead to difficult bugs in our code. First, one should probably always use semicolons *explicitly*, not relying on the parser to do it for you. Secondly, be aware that line breaks can sometimes be the cause of this if the parser was expecting something else and we must try to prevent this - e.g., always placing brackets on the same line as keywords appear. 

**Whitespace**  
Invisible characters that create literal *space* in your code - tabs, carriage returns, spaces, etc. In general, JS parsers are very liberal in how they let us use whitespace.

**Immediately Invoked Functions (IIFEs)**  

```javascript
// function statement
function greet(name) {
  console.log('Hello ' + name);
}
greet('John'); 

// function expression
var greet = function(name) {
  console.log('Hello ' + name);
}
greet('John');

// IIFE
var greet = function(name) {
  return 'Hello ' + name;
}('John');

console.log(greet);

// also an IIFE
(function(name) {
  var greeting = 'Hello ';
  console.log(greeting + ' ' + name);
}('John')); 
```

**Note** wrapping the function in parens tricks the parser into thinking that it is an expression rather than a statement. This type of form is used in many major frameworks and libraries. 

**IIFEs and Safe Code**  
The main benefit of the IIFE is that it doesn't pollute the global namespace as all variables are completely contained in the execution context of the IIFE. It doesn't crash into, nor is it interfered with by any other code. This is what makes IIFEs so useful for libraries and frameworks. Additionally, if one does want to have access to the global environment, one can just pass that into the function as a parameter - this works because objects are passed by reference and one can then append (intentionally) any properties or functions(methods) onto that object.

**Closures**  

```javascript
function greet(whattosay) {
  return function(name) {
    console.log(whattosay + ' ' + name);
  }
}

var sayHi = greet('Hi');
sayHi('John');
```

A **closure** is an inner function that has access to the outer enclosing function's variables and scope chain. Indeed, the closure has access to three links in the scope chain: it's own scope (the variables defined in it's own set of curly brackets), the outer (enclosing) functions scope, and the global scope.
**Note** how the function returned from the greet function still has access to the 'whattosay' argument/variable even though the greet function is no longer on the stack. This is possible because the JS engine preserves the scope for all the variables that were in scope at the time the closure was created - i.e., the lexical environment of the function when it was declared.
**Free Variables** are just variables that exist beyond a function that you have access to - the variables in the outer scope chain that a closure is accessing are free variables.
**Note** how useful IIFEs can be when iterating over closures - i.e., to capture the value of the iteration at any given point by creating an execution context which captures that value (see the count to ten in 1 sec interval problem).

**Function Factories**  

```javascript
function makeGreeting(language) {
  
  return function(firstname, lastname) {
      
    if (language === 'en') {
      console.log('Hello ' + first name + last name);
    }

    if (language === 'es') {
      console.log('Hola ' + first name + last name);
    }

  }

}

var greetEnglish = makeGreeting('en');
var greetSpanish = makeGreeting('es');
```
See how the inner function that is returned from makeGreeting utilizes the language parameter that gets wrapped up in the closure of makeGreeting's execution stack. This is a function factory in that each time makeGreeting is called we get a new execution context that closes in the arguments that we provided during the invocation.

**Closures and Callbacks**
  
```javascript
function sayHiLater() {

  var greeting  = 'Hi!';
      
  setTimeout(function() {
      console.log(greeting);
  }, 3000);

}

sayHiLater();
```

Here we can see how the language makes use closures with callbacks. Similarly, both JS and jQuery utilize closures and callbacks for event handlers.

**Callback Functions**  
A function you give another function to run when it is finished executing. So the function you called (invoked) *calls back* by running the callback function when it finishes. 

```javascript
function finished(callback) {
  var done = 'DONE!';
  callback(done);
}
finished(function(done) {
  console.log(done);
})
```

**call(), apply() and bind()**  
Remember that functions in JS are just a special class of objects. All functions have both a (optional) *name* property and an invocable *code* property. They also have access to 3 additional methods as well: *call, apply and bind*. These all have to do with what the 'this' variable refers to and arguments given to the function as well.

**call()**  
Simply allows us to invoke a function in much the same way that parens would, but it also allows us to specify what the 'this' variable refers to (first parameter) followed by the arguments (2nd parameter on).

```javascript
somefunction.call(someObj, arg1, arg2, ...)
```

**apply()**  
Almost the same thing as the *call()* method, except that the arguments list must be provided as an array...

```javascript
someFunction.apply(someObj, [arg1, arg2, ...])
```

**bind()**  
Creates a copy of whatever function you are calling it on and binds the 'this' variable to whatever object you give as an argument to it.

```javascript
someFunction.bind(someObj)
```

Examples:
```javascript
var person = {
  firstname: 'John',
  lastname: 'Doe',
  getFullName: function(){
      var fullname = this.firstname + ' ' + this.lastname;
      return fullname; 
  }
}

var logName = function(lang1, lang2) {
  console.log('Logged: ' + this.getFullName());
  console.log('Arguments: ' + lang1 + ' ' + lang2);
  console.log('------------------');
}

logName.call(person, 'en', 'es');
logName.apply(person, ['en', 'es']);

var loggedPerson = logName.bind(person);

loggedPerson('en', 'es'); 

// Function Borrowing

var person2 = {
  firstname: 'Jane',
  lastname: 'Doe'
}

console.log(person.getFullName.apply(person2));

//Function Currying

function multiply(a, b) {
  return a * b;
}

var multiplyByTwo = multiply.bind(this, 2);
console.log(multiplyByTwo(4));

var multiplyByThree = multiply.bind(this, 3);
console.log(multiplyByThree(4));
```

**Function Borrowing**  
As long as one has similar property names (so that the methods will work) one can 'borrow' methods from one object to be used by another - in the case of JS by using call( ) or apply( ).

**Function Currying**  
Creating a copy of a function but with some preset parameters - very useful in mathematical situations.

**Functional Programming**  
Complex(ish)... checkout:  
[Eloquent Javascript](http://eloquentjavascript.net/1st_edition/chapter6.html)  
and  
[What is Functional Programming](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-functional-programming-7f218c68b3a0)  
Once you have a handle on the concepts, checkout:  
[underscoreJS](http://underscorejs.org)  
(try going through the annotated source code - open source education)

Example of functional programming in JS:

```javascript
function mapForEach(arr, fn) {
  var newArr = [];
  for(var i = 0; i < arr.length; i++){
    newArr.push(fn(arr[i]));
  }
  return newArr;
}
var arr = [1, 2, 3];
console.log(arr);

var arr2 = mapForEach(arr, function(item) {
  return item * 2;
});
console.log(arr2);

var arr3 = mapForEach(arr, function(item) {
  return item > 2;
});
console.log(arr3);

function checkPastLimit(limiter, item) {
  return item > limiter;
}

var arr4 = mapForEach(arr, checkPastLimit.bind(this, 1));
console.log(arr4);

function checkPastLimitSimplified(limiter) {
  return function checkPastLimit(limiter, item) {
  return item > limiter;
  }.bind(this, limiter);
}

var arr5 = mapForEach(arr, checkPastLimitSimplified(2));
console.log(arr5);
```

## Objects

**Object-oriented JS and Prototypal Inheritance**  

**Inheritance**  
One object gets access to another object's properties and methods.

**Classical Inheritance**  
Somewhat verbose and doesn't really exist in JS. In other programming languages it is based on *classes* which, despite some syntactic sugar in ES6, is not possible in JS. *A class is like a blueprint - a description of the object to be created*. Classes inherit from classes and create sub-class relationships: hierarchical class taxonomies. *Instances* are typically instantiated via constructor functions with the '*new*' keyword.

**Prototypal Inheritance**  

**The Prototype**  

```javascript
var person = {
  firstname: 'Default',
  lastname: 'Default',
  getFullName: function() {
    return this.firstname + ' ' + this.lastname;
  }
}

var john = {
  firstname: 'John',
  lastname: 'Doe'
}

// Don't ever do this!!! for demo only!

john.__proto__ = person;
console.log(john.getFullName());
```

**The Prototype Chain**  
Everything in JS is either an object or a primitive. All objects, with the exception of the base object itself, have a prototype. If a property or method cannot be found on the object that one is referencing, the JS engine will check the object's prototype for it and if it is not found will check its prototype's prototype and so on until it eventually arrives at the base object which has no prototype and the prototype chain ends. 

**Reflection and Extend**  

**Reflection**  
An object can look at itself, listing and changing its properties and methods.  

**extend**  
say for example in underscore js... extend takes one object and adds to it all the enumerated properties of some other object and it's prototypes, excluding the base object. Note that this is different from the setting up those additional objects in the prototype chain as their properties and methods are actually added as properties to the target object rather than simply being available in the prototype chain. **Note**: this is not the same as the ES6 'extends' which is used to set the prototype for an object.

**Building Objects**  

**Function Constructors and the 'new' Keyword**  

```javascript
function Person(firstname, lastname) {

  console.log(this);
  this.firstname = firstname;
  this.lastname = lastname;
  console.log('This function is invoked!');

}

var john = new Person('John', 'Doe');
var jane = new Person('Jane', 'Doe');

console.log(john); 
console.log(jane);
```

The 'new' keyword first creates an empty object and then invokes the constructor function (in this case the Person function); it also, and importantly, changes the reference of the 'this' keyword to point not to the global object, but rather to the empty object it just created. Note that as long as the function doesn't explicitly return something, the newly created object will be returned. *Constructor functions are just normal functions* that are used to construct objects - The 'this' variable points to *a new empty object and that object is returned from the function automatically*. The 'new' keyword actually creates the new object, the function constructor adds properties and methods to it. 

**Function Constructors and '.prototype'**  
Remember that functions are just objects. Every function has a 'prototype' property. It just sits there doing nothing until one uses the 'new' keyword when invoking the function - this begins life as an empty object and is accessible to us. Then any objects created by this function will point to this empty object as their prototype. (Bear in mind that this is not the function's prototype, but the prototype of any objects created by that function.) In relation to the above code, we could then do something like this:  

```javascript
Person.prototype.getFullName = function() {
  return this.firstname + ' ' + this.lastname;
}
```

Part of the power of this is that we can then add methods and properties on the fly by just adding them to *Object.prototype.somePropOrMethod*. This is extremely useful for a number of additional reasons as well. For example, and you will often see this in good JS code, it is good practice to place properties on the object (at least the ones that vary object to object), but methods are added to the prototype which, while giving all objects that have it in their prototype chain access to it, it uses less memory than if it were attached to each individual object repeatedly (imagine if in the above example we had 1,000,000 instances of people). 

**Dangerous Aside**  
Because function constructors are just normal functions, we can invoke them without using the 'new' keyword. However, since the function itself does not return anything - it is the use of the 'new' keyword that causes the function to return the newly created object - any objects we think we are creating will actually point to 'undefined' as their value. This leads to the convention of always naming function constructors with a capital letter and then we can always infer that functions beginning with a capital *must* be invoked with the 'new' keyword - should we make the aforementioned mistake, this will greatly aid us in debugging. There are new methods that work better than using function constructors.

**Built-in Function Constructors**  
Number(), String(), Boolean(), Date(), etc.,. Note that when using these you are actually creating objects and that these then get various method attached to them. Actually, the JS engine will even wrap our primitives in the associated objects thus allowing us to utilize various methods on them - e.g., '.length' on a string. Note that we can also add methods and properties to these as well. So for example we could add a method to all numbers in our code simply by doing something like this:

```javascript
Number.prototype.greaterThan100 = function() {
  return this > 100;
}
var a = 12;
console.log(a.greaterThan100());
```

**Dangerous Aside**
  
```javascript
var a = 3; 
var b = new Number(3);
console.log(a == b) // returns true;
console.log(a === b) // returns false;
```

**Note** The reason why the above happens is that when we call one of the built in function constructors with the 'new' keyword, we are in fact creating an object rather than a true primitive, and hence strict equality fails. Indeed, at least for primitives, it is almost always preferable to use the literal rather than the function constructor. That said, many libraries and frameworks use the technique of adding methods onto built in prototypes and it can be quite powerful.

**Dangerous Aside: Arrays and 'for...in'**  
Remember that arrays are essentially objects in JS and, consequently, it is possible to use a 'for... in' loop to enumerate their properties and associated values. However, this type of loop will, in addition to the elements of the array, iterate over any methods and/or properties that have been explicitly added to the array prototype as well. It is almost always better to a regular 'for' loop to iterate over an array.

**Oject.create() and Pure Prototypal Inheritance** 
 
```javascript
var person = {
  firstname: 'Default',
  lastname: 'Default',
  greet: function() {
      return 'Hi ' + this.firstname;
  }
}

var john = Object.create(person);
john.firstname = 'John';
john.lastname = 'Doe';
console.log(john);
```

... just embrace pure prototypal inheritance

**ES6 and Classes**
  
```javascript
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
  
  greet() {
    return 'Hi ' + firstname;
  }
}
var john = new Person('John', 'Doe');
console.log(john);
```

This is just syntactic sugar and does not constitute real classes - it is still just an object. Everything is the same as before under the hood and Object.create is still the pure JS way to do things.

**Note** if you want to set one class as the prototype of another with this new syntax:
  
```javascript
class InformalPerson extends Person {
  constructor(firstname, lastname) {
    super(firstname, lastname);
  }

  greet() {
    return 'Yo ' + firstname;
  }
}
```

**extends** sets the prototype for the class
**super** just runs the constructor for the prototype

## Odds and Ends  

**typeof, instanceof and figuring out what something is** 
 
```javascript
var a = 3;
console.log(typeof a);

var b = 'Hello';
console.log(typeof b);

var c = {};
console.log(typeof c);

var d = [];
console.log(typeof d); // weird!
console.log(Object.prototype.toString.call(d)); // better!

function Person(name) {
  this.name = name;
}

var e = new Person('Jane');
console.log(typeof e);
console.log(e instanceof Person);

console.log(typeof undefined); // makes sense
console.log(typeof null); // a bug in the language forever

var f = function() {};
console.log(typeof f);
```

**Strict Mode**  
[MDN strict mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)

**Check out the code source for jQuery - Free Open Source Education**  

**Method Chaining**  
Calling one method after another and each method affects the parent object. So...

```javascript
obj.method1().method2();
```

... each method ends up with a 'this' variable pointing at obj. When we look at the jQuery library, we see that method chaining is accomplished by returning 'this' at the end of each method - by returning 'this' we can then call additional methods that exist on what 'this' points too.

**Typescript, ES6 and Transpiled Languages**  
**Transpiled**  one language converted into another - in this case the written language never really runs anywhere, but instead are processed by 'transpilers' and converted into JS. *See: Typescript, Traceur, etc.* It is fine to use these insofar as we have a (deep) understanding of the language they are being converted into. 

**[ES6]( https://github.com/lukehoban/es6features)**