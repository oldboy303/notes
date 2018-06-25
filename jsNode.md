NODEJS

## V8 The Javascript Engine  

**Processors, Machine Language and C++**  
Processors interpret instructions in whatever machine language they are built to use - IA-32, x86-64, ARM, MIPS, etc. These are very low level languages and every program/instruction that is run on a computer is eventually compiled to one of these *machine code* languages that communicate directly with the processor.

NodeJS is written in C++. This is the case because the V8 Engine is written in C++.

**_JS Engine_** a program that compiles our JS code into machine code so that the processor can execute our instructions.

**Adding Features to JS**  
The whole point of Node is that it adds a bunch of features to JS that allow the language to function, usefully, as a server-side programming language - manipulating files, running a server, etc.

## The Node Core  

**What does JS need to manage a server?**  
* Better ways to organize our code into reusable pieces
* Ways to deal with files
* Ways to deal with databases
* The ability to communicate over the internet
* The ability to accept requests and send responses over the internet (in a standard format)

Node provides all of the above by allowing us to use C++ functionality - C++ is a much more fully featured programming language that allowd us to do all of the aforementioned and which is not provided for in JS.

Consequently, if we look at the source code for Node, we see that it has both a *C++ core* and a *JS core*.

In the JS core we see that some merely wraps C++ functionality and some is pure JS code that creates new functionality within JS itself.

## Modules, Exports and Require  

**Modules**  
A reusable block of code whose existence does not accidentally impact other code (JS did not have this before ES6).

**CommonJS Modules**  
An agreed upon standard for how code modules should be structured.

**_Checkout JS Understanding the Weird Parts for descriptions of objects, prototypal inheritance, IIFEs, by value and by reference, first-class functions, JSON, etc - all of which NodeJS uses to great effect in creating modules, exports and require_**

**_require_**  
is a function that you pass a *path* too. (Note: Node will automatically look for a file with .js extension. If it does not find one, it will look for a folder of the same name and, if it finds one, will try to find a run a file named 'index.js'. In that folder we can create a number of modules in seperate files and then just 'require' them into the index.js file. We can then in some app file just require the folder with the index.js file and c=gain access to all the modules therein.)

**_module.exports_**  
is what the *require* function returns. This works because *your code is actually wrapped in a function* that is given these things as function parameters (*dirname, filename, exports, path, etc)... see the Node JS core code to see this code for yourself.

**Module Patterns**  
Overwriting the exports object: 

```javascript
module.exports = function() {
  console.log('Hello World!");
}
```

Adding to the exports object: 

```javascript
module.exports.greet = function() {
    console.log('Hello World!');
}
```

Replacing the exports object: 

```javascript
function Greetr() {
    this.greeting = 'Hello World!!!';
    this.greet = function(){
        console.log(greeting);
    }
}

module.exports = new Greetr();
```

Note: Node caches modules and because of this we can run into issues with passing by reference when we require the same file multiple times, even across multiple js files.

Return the function constructor itself:

```javascript
function Greetr() {
    this.greeting = 'Hello World!!!';
    this.greet = function(){
        console.log(greeting);
    }
}

module.exports = new Greetr;
```

… we would then have to call the function constuctor in whatever file this module is required into. Doing it this way prevents us from running into problems due to passing by reference.

Closure(ish) exports:

```javascript
var greeting = ‘Hello World!!!’;

function greet() {
    console.log(greeting);
}

module.exports = {
    greet: greet
}
```

Note: we are only exposing the greet function and all variables, and indeed any other logic we might add outside of the function, remains private. This is also known as the…

**_Revealing Module Pattern_**  
exposing only the properties and methods you wish via the returned object - a very common and clean way to structure and protect code within modules.

**Note**: that we can use the shorthand ‘exports’ instead of ‘module.exports’, but only if we mutate it rather than set it equal to something. This is because, in the Node framework, ‘exports' is set equal to ‘module.exports'. Thus, when we set ‘exports' equal to something else, we break the reference pointing to the same object that ‘module.exports' points to in memory and, as ‘require’ returns ‘module.exports’, require ends up pointing to a (still) empty module.exports object. Mutating the object, however, does not break the reference. Probably it is best just to use **module.exports** so as to reduce our cognitive load. 

**Requiring Native Modules**  
Remember that ‘require’, in the Node core code, checks to see if something is a native in the core code. Check the Node API docs to see all the native modules. 

**Modules and ES6**  
Note that modules have finally made an appearance in ES6 - i.e., 

```javascript
import * as greetr from ‘greet’;
```

 (assuming we have a greet.js file). Node now supports this, but Node modules won’t be going anywhere anytime soon. 

## Events and the Event Emitter  

**Events**  
Something has happened in our app that we can respond to - in Node we are actually talking about two kinds of events...

**System Events**  
These come from the C++ core via libuv - these are events like *I finished reading a file, received data from the internet, the file is open, etc*.

**Custom Events**  
These exist within the JS core - these are events that we can create in JS and designate responses to and arise from code that creates what is called the '*event emitter*'.

**Event Listener**  
The code that responds to an event - in the JS case, the listener will be a function. 

**Magic Strings**  
A string that has some special meaning in our code - bad because it means that a typo in our code can cause a bug and it is difficult for our tools to find such bugs. Note how the emitter relies on these and how we can resolve some of these concerns by using a config file.

The **Event Emitter** in Node basically works by creating an object and attaching properties to it (event types) which point to arrays of functions (listeners). When an event of a certain type is ‘emitted’, the Event Emitter then loops through the array of listeners, running each function in turn; if there is only a single function/listener, it is just called immediately. Check the Node source code to see how they set it up.

Note how we can use the Node events and util to create our own event emitting objects:

```javascript
var EventEmitter = require('events');
var util = require('util');

function Greetr() {
    EventEmitter.call(this); // To capture any props that this constructor adds to the created object
    this.greeting = 'Hello World';
}

util.inherits(Greetr, EventEmitter);

Greetr.protoype.greet = function() {
    console.log(this.greeting);
    this.emit('greet'); // because we are now inheriting from the EventEmitter
}

var greetr1 = new Greetr();

greetr1.on('greet', function() {
    console.log('Someone Greeted!');
});

greetr1.greet();
```

**Note**: uitl.inherits just connects the prototypes, but does not actually run the super constructor and, hence, those properties it attaches to the new object will not be present - hence the line of code above ('EventEmitter.call(this)').

## Asynchronous Code, libuv, The Event Loop, Streams, Files, and more  
**Asynchonous**  
More than one process running at a time - Node processes asynchronously, V8 does not. In other words, JS is synchronous and NodeJS is asynchronous - while the V8 engine, which is embedded in Node, is running a process, Node might and usually is running other processes (via the C++ core).

**Callbacks**  
a function passed to another function which we assume will be invoked at some point in the future - the first function 'calls back' invoking the function you give it when it is done doing its work.

Bearing the aforementioned in mind, an **_event loop_** is running utilizing *libuv* in the C++ core of Node. When an event occurs (a file is done reading, an http request returns, etc) it is added to an **_event queue_** and those events are then processed and a callback is executed, often involving some JS code which is handed off to the V8 engine embedded in Node and processed synchronously. Referring to these processes, we often see phrases like "Event driven non-blocking I/O in V8 Javascript".

**Non-Blocking**  
Doing other things without stopping your programming from running - this is made possible by the C++ side of Node doing things asynchronously.

**Buffers**  
A temporary holding place for data being moved from one place to another - intentionally limited in size.

**Stream**  
A sequence of data made available over time - pieces of data that eventually combine into a whole. We often process this data continually... think of streaming a movie online versus downloading all of it and then watching it.

**Character Sets**  
Because all data that computers deal with is at the lowest level just binary data/numbers, all other data must first be converted into numbers. Hence, character sets are just different encodings giving us representations of characters as numbers (which are later converted to binary) - **_Unicode_** and **_ASCII_** are character sets. 

**Character Encoding**  
how characters are stored in binary - the numbers (or *code points*) are converted and stored in binary. Basically, how many bits each number has available for its representation. For example, UTF-8 provides 8 bits for character encodings - in a stream of binary data, using UTF-8, every eighth digit is a new number.

Note that JS does not have many native ways to deal with binary data and part of what Node provides us are better ways to handle binary data, streams, etc.

* Checkout buffers in Node and ES6 Typed Arrays

**Error-First Callbacks**  
Callbacks take an error object as their first parameter - null if there is no error, otherwise will contain an object defining the error. This is a standard so that we know in what order to place our parameters for our callbacks.

**Files and FS**  

```javascript
// must have a greet.txt file in the home directory
var fs = require('fs');

var greet = fs.readFileSync(__dirname + '/greet.txt', 'utf8');

console.log(greet);

var greet2 = fs.readFile(__dirname + '/greet.txt', 'utf8', function(err, data) {
    // usually some error checking here
    console.log(data);
});

console.log('Done!');
```

**Note**:  
The above asynchronous code, while better than synchronous, could still cause us some problems - namely, the buffer sits on the heap and if enough people are using the app and/or the buffer is large enough we could run into performance issues because of memory usage...

**Chunks**  
A piece of data being sent in a stream - data is split into 'chunks' and streamed.

**Streams**  
note that streams in Node inherit from and, consequently, are a type of *event emitter*... Streams are also:

**Abstract (Base) Class**  
A type of constructor you never work with directly, but inherit from - we create custom objects which inherit from the base class.

**Readable and Writable Streams**  
```javascript
// must have a greet.txt and greetcopy.txt files in the home directory
var fs = require('fs');

var readable = fs.createReadStream(__dirname + '/greet.txt', {
    encoding: 'utf8',
    highWaterMark: 16 * 1024
});

var writeable = fs.createWriteStream(__dirname + '/greetcopy.txt');

readable.on('data', function(chunk) {
    console.log(chunk);
    writeable.write(chunk);
});
```

**Note**:  
how we have now limited the buffer size and no longer need to read the whole file into the buffer.

**Pipes**  
connecting two streams by writing to one stream what is being read from another - in Node you pipe a readable stream to a writeable one...

```javascript
// must have a greet.txt, greet.txt.gz and greetcopy.txt files in the home directory
var fs = require('fs');
var zlib = require('zlib');

var readable = fs.createReadStream(__dirname + '/greet.txt'});

var writeable = fs.createWriteStream(__dirname + '/greetcopy.txt');

var compressed = fs.createWriteStream(__dirname + '/greet.txt.gz');

var gzip = zlib.createGzip();

readable.pipe(writeable);

readable.pipe(gzip).pipe(compressed); //method chaining with pipe
```

**Method Chaining**  
A method returns an object so we can keep calling more methods - sometimes it returns the parent object (technically called 'cascading') and sometimes it returns a new object. 

## HTTP and being a Web Server  

**TCP/IP**  
*Internet Protocol* basically the unique identifier/address of every computer connected to the internet. A *socket* is basically opened between two IP addresses and then there are various protocols for sending data across the socket - *HTTP* for webpages, *FTP* for files, *SMTP* for email, etc - and which determine how the data is structured. That said, the actual lower level protocol that deals with the actual transmission of that data is *Transmission Control Protocol(TCP)* and breaks that data into *packets*. Think of these as basically *chunks* flowing through a *stream* and that is more or less how Node treats them. 

Note that we typically think of Node as a web server, but because it has access to TCP/IP, it could of course be used to build email servers, file servers, etc.

**Port**  
once a computer receives a packet, how it knows what program to send it to - when a program is setup on the operating system to receive packets from a particular port, it is said that the program is 'listening' to that port. A port is really a mapping of a number to a particular program on the computer. Note that an IP address plus a port number is know as a *socket address*.

**HTTP**  
HyperText Transfer Protocol. A set of rules (and a format) for data being transferred on the web - it's a format of various data being transferred via TCP/IP

**MIME Type**  
a standard for specifying the type of data being sent - *Multipurpose Internet Mail Extensions (MIME)*. Examples: *application/json, text/html, image/jpeg*, etc.

**Simple HTTP Server in Node**  

```javascript
const http = require('http');

http.createServer(function(req, res) {

    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World!\n');

}).listen(1337, '127.0.0.1')
```

**Template**  
text designed to be the basis for some final text or content once it is processed - there is usually some specific template language so that the template system knows how to replace the placeholders with real values - e.g., Handlebars, Pug, etc.

**index.htm**  

```javascript
<!DOCTYPE html>
<html>
    <head></head>
    <body>
        <h1>{Message}</h1>
    </body>
</html>
```

**app.js**  

```javascript
const http = require('http');
const fs = require('fs');

http.createServer(function(req, res) {

    let html = fs.readFileSync(__dirname + '/index.htm', 'utf8');
    let message = 'Hello World!!!';
    html = html.replace('{Message}', message);

    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end(html  );

}).listen(1337, '127.0.0.1');
```

**Note**: if we don't want to manipulate the data, we could just pipe it over to the response...

```javascript
const http = require('http');
const fs = require('fs');

http.createServer(function(req, res) {

    res.writeHead(200, { 'Content-Type': 'text/html' });
    fs.createReadStream(__dirname + '/index.htm').pipe(res);

}).listen(1337, '127.0.0.1');
```

**API**  
 a set of tools for building a software application - *Application Programming Interface (API)*. On the web the tools are usually made available via a set of URLs which accept and send only data via HTTP and TCP/IP.

 **Endpoint**  
 one URL in a web API - sometimes that endpoint (URL) does multiple things by making choices based on the HTTP request headers. (Most popular format for data currently is JSON). 

 **Serialize**  
 translating an object into a format that can be stored or transferred - *JSON, CSV, XML*, and others are popular. Deserialize is the opposite process of converting the formatted data back into an object. 

 **Routing**  
 mapping HTTP requests to content - whether the actual files exist on the server or not (i.e., in a DB for example).

 **Super basic routing...**  

 ```javascript
 const http = require('http');
const fs = require('fs');

http.createServer(function(req, res) {

    if(req.url === '/') {
        res.writeHead(200, { 'Content-Type': 'text/html' });
        fs.createReadStream(__dirname + '/index.htm').pipe(res);
    } 
    else if(req.url ==='/api') {
        res.writeHead(200, { 'Content-Type': 'application/json'});
        var obj = { firstname: 'John', lastname: 'Doe' };
        res.end(JSON.stringify(obj));
    }
    else {
        res.writeHead(404);
        res.end();
    }

}).listen(1337, '127.0.0.1');
```

## NPM - Node Package Manager  
The largest open source repository in history...

**Package**  
code... managed and maintained with a package management system.

**Package Management System**  
a system that automates the installing and updating of packages. It deals with the getting the version you have or need and managing dependencies. 

**Dependencies**  
code that another set of code depends on to function. Think of how so much of Node depends (or inherits from) on the Event Emitter code - if the Event Emitter code was external to Node and you had to install it for Node to function properly, that would be a dependency.

**Versioning**  
specifying what version of code something is... so that others can track whether a newer version has come out, whether there are new features to look for, whether there are *breaking changes*, etc. This is critical in managing dependencies.

**Semantic Versioning**  
just implies that the way we version our code carries some meaning.   Checkout [this](http://www.semver.org)  
For example:   
*Major.Minor.Patch*  
Version 1.7.2  
*Patches* - bug fixes no breaking changes.  
*Minor* (update) - new features were added, still no breaking changes  
*Major* - Big changes. Your code might break. 

## Express  

**Environment Variables**  
global variables specific to the environment (server) that our code is living in - different servers can have different variable settings and we can access those values in code. 

**HTTP Methods/Verbs**  
specifies the type of action the HTTP request is trying to make - *POST, GET, PUT/PATCH, DELETE (corresponds roughly to CRUD operations).

**Express takes advantage of these methods for routing**

```javascript
app.get('/', function(req, res) {
    res.send('<html><head></head><body><h1>Hello World!</h1></body></html>');
});
```

Note that express also checks what we are sending back and sets the content(MIME) type for us, among other things, in the header.

**Routing in Express**  
First, checkout expressjs.com and look at the docs for all the possibilities for routing - params, queries, pattern matching, etc. Here is a params example:

```javascript
app.get('/person/:id', function(req, res) {
    res.send('<html><head></head><body><h1>Person: ' + req.params.id + '</h1></body></html>');
});
```

**Middleware**  
code that sits between two layers of software - in the case of Express, sitting between the request and the response. Assuming there is a 'public' folder with a 'styles.css' file...

```javascript
app.use('/assets', express.static(__dirname + '/public'));

app.get('/', function(req, res) {
    res.send('<html><head><link href=assets/style.css type=text/css rel=stylesheet/></head><body>                                  <h1>Hello World!</h1></body></html>');
});
```

**Note**:  
*app.use()* just takes a route and a function which takes *req, res, and next parameters*:

```javascript
app.use('/', function(req, res, next) {
    // do some work here;
    next(); // calls the next middleware
});
```

**Templates and Templating Engines**  
Checkout various template engines - jade/pug, handlebars, ejs, etc.
Set the view engine in Express with:

```javascript
app.set('view engine', 'ejs'); // the second parameter is just the file extension used for the views

//then on the route use:
res.render(<<filename>>);
```

**Querystring and POST Parameters**  
Handling query strings are probably the easiest as Express already parses the query string from the header for us and we can access any values simply with *req.query.whateverTheNameOfTheValueIs*

```javascript
req.query.id //for example
```

On POST requests, it is somewhat more difficult. First we need to install *middleware* to parse the body of the POST request (*body-parser* is th go-to package for this).

```javascript
var express = require('express')
var bodyParser = require('body-parser')
 
var app = express()
 
// parse application/x-www-form-urlencoded
app.use(bodyParser.urlencoded({ extended: false }))
 
// parse application/json
app.use(bodyParser.json())
 
app.use(function (req, res) {
  res.setHeader('Content-Type', 'text/plain')
  res.write('you posted:\n')
  res.end(JSON.stringify(req.body, null, 2))
})
```

... or route-specific...

```javascript
var express = require('express')
var bodyParser = require('body-parser')
 
var app = express()
 
// create application/json parser
var jsonParser = bodyParser.json()
 
// create application/x-www-form-urlencoded parser
var urlencodedParser = bodyParser.urlencoded({ extended: false })
 
// POST /login gets urlencoded bodies
app.post('/login', urlencodedParser, function (req, res) {
  if (!req.body) return res.sendStatus(400)
  res.send('welcome, ' + req.body.username)
});
 
// POST /api/users gets JSON bodies
app.post('/api/users', jsonParser, function (req, res) {
  if (!req.body) return res.sendStatus(400)
  // create user in req.body
});
```

Note that the above code includes both *urlencoded and JSON* parsers.

**RESTful APIs**  
REST is an architectural style for building APIs. Stands for: *Representational State Transfer*. Essential to the idea is that HTTP verbs and URLs *mean* something. 

Sample RESTful API:

```javascript
app.get('/api/person/:id', function(req, res) {
    // get a person from the database
});

app.post('/api/person', function(req, res) {
    // insert a person into the database
});

app.delete('/api/person/:id', function(req, res) {
    // delete a person from the database
})
```

**Structuring an App**  
Check out the Express generator for one type of structure. Alternately, check out MVC as a possible structure - moving the view part of the model onto the client with Angular or React. 

## Javascript, JSON and Databases  

Think about how we translate data into JS objects - i.e., we might convert tabular data of names and addresses into a JS array where each item (person and address) in the array is a JS object. Particularly effective when dealing with SQL/Relational databases.

NO SQL databases are somewhat more simple to deal with.
Checkout **MongoDB, Mongoose, and MongoLab**

## The MEAN Stack: MongoDB, Express, Angular, and Node

**Stack**   
the combination of all technologies use to build a piece of software - db, server-side, client-side, and whatever else. 

Note that the popularity of the mean stack arises from the prevalence JS throughout the stack - Mongo is not really JS, but it does store data in a format that JS can work with easily. 


**DOM**  
*Document Object Model* The structure browsers use to store and manage webpages. Browsers give the JS engine the ability to manipulate the DOM. The underlying structure of the DOM is basically a tree-like structure of C++ objects. 