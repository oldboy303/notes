ANGULAR (1.5ish)  

## Model, View, Whatever  

**MV\***    
Model View and the \* (controller, view model, whatever) is what *binds* the model to the view and vice versa.

**HTML Custom Attributes**  
Angular takes advantage of the fact that one can access custom attributes in the DOM (Document Object Model) in the browser to access elements, bind elements to models, etc. 

```html
<h1 some-custom-attribute="xxxx">Hello World!</h1>
<!-- or if we want to be HTML compliant --> 
<h1 data-some-custom-attribute="xxxx">Hello World!"</h1>
```

## Services and Dependency Injection  

**Dependency Injection**  
giving a function an object - rather than creating an object inside a function, you pass it to the function. 

```javascript
var Person = function(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
};

function logPerson(person) {
    console.log(person);
}

var john = new Person('John', 'Doe');
logPerson(john); // this is the "dependency injection"
```

Admittedly this is a super simple example, but at it's most basic level this is all dependency injection is. Angular uses this idea across the whole framework (controllers, scope, etc) to create and enforce a stable architecture. 

**The Scope Service ($scope)**  
basically *$scope* is an object defined by Angular (all Angular services begin with a '$') and is available throughout the app. Essentially, this object is what will hold all the data that is shuttled between the *controller* and the *view* of our app. All we have to do is feed it (or rather 'inject' it) to any function that needs it. 

**How Angular does Dependency Injection**  
Angular, then, achieves its version of dependency injection by parsing functions parameters and anywhere it recognizes a parameter name  -e.g., like '$scope' - it will create that object (if it does not already exist) and provide it to the function as an argument.

```javascript
var myApp = angular.module('myApp', []);

myApp.controller('mainController', function($scope) {
    console.log($scope);
})

var searchPeople = function(firstname, lastname, $scope, occupation, age) {};

console.log(angular.injector().annotate(searchPeople)); 
// returns an array of the parameters as strings for searchPeople... ['firstname', 'lastname', '$scope', 'age']
```

... a more realistic example...

```javascript
var myApp = angular.module('myApp', []);

myApp.controller('mainController', function($scope, $log, $filter) {
    $scope.name = 'John';
    $scope.formattedName = $filter('uppercase')($scope.name);
    
    $log.info($scope.name);
    $log.info($scope.formattedName);
});
```

**Requiring Modules**  
in addition to the Angular core library, we might also add numerous modules to our app - checkout out the [docs](https://docs.angularjs.org/api#angularjs-modules) for a list of these. Here is an example:  

index.html

```html
<!-- core angular -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.6.5/angular.js"></script>

<!-- the angular messages module -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.6.5/angular-messages.min.js"></script>
```

app.js

```javascript
var myApp = angular.module('myApp', ['ngMessages']); // messages added to app dependencies

myApp.controller('mainController', function($scope, $log, $filter) {
   
});
```

**Minification**  
shrinking the size of files for faster downloads - we normally add '.min' to the file name so that we can tell the difference... So 'file.js' becomes 'file.min.js'. Note that when a file is minified its variables and function parameters are often reduced to single characters to save space: 

```javascript
function($scope, $log) {
    log.info($scope);
}
```

becomes...

```javascript
function(a,b){b.info(a)}
```

This presents a serious problem for angular apps, however, in that angular is relying on the parsed parameter names/strings to recognize services/dependency injections and will, with the minified version, throw injector error(s). The solution to avoid this is to pass an array with the parameters/dependencies wrapped in quotes (strings aren't affected by minification) and the function as the last element of the array:

```javascript
var myApp = angular.module('myApp', []);

myApp.controller('mainController', ['$scope', '$log', function($scope, $log) {
    $log.info($scope);
}]);
```

Angular basically uses the elements preceding the function in the array to interpret the minified parameters - note that, consequently, the order of the elements of the array matter here.

## Data Binding and Direction  

**Scope and INterpretation**  
interpolation is just creating a string by combining other strings and placeholders. Hence:  

```javascript
var name = 'San';

console.log('My name is ' + name); // the output of this line is interpolated
```

Angular constantly checks the scope and the view (*dirty checking*) and can update the view whenever something changes...

index.html

```html
<div ng-controller="mainController">
    <h1>Hello {{ name + '. How are you?'}}</h1>
</div>
```

app.js  

```javascript
var myApp = angular.module('myApp', []);

myApp.controller('mainController', ['$scope', '$timeout', function($scope, $timeout) {
    $scope.name = 'San';
    $timeout(function() {
        $scope.name = 'Everybody';
    }, 3000);
}]);
```

This will first output 'Hello San. How are you?' and then after three seconds when the name variable on scope changes it will output 'Hello Everybody. How are you?'. Take note of how we access the variable on $scope in the html with '{{}}' and how we specify what part of the html has access to it with 'ng-controller'.

**Directives**  
an instruction to angular to manipulate a piece of the DOM - e.g., 'add a class', 'hide this', 'create this', etc. 'ng=app' and 'ng-controller' are directives. We also use directives to set up our *two-way data binding* in angular:  

index.html

```html
<div ng-controller="mainController">
    <div>
        <label>What is your twitter handle?</label>
        <input type="text" ng-model="handle">
    </div>
    <hr>
    <h1>twitter.com/ {{ handle }}</h1>
    <hr>
    <h1>twitter.com/ {{ lowercaseHandle() }}</h1>
</div>
```

app.js

```javascript
var myApp = angular.module('myApp', []);

myApp.controller('mainController', ['$scope', '$filter', function($scope, $filter) {
    $scope.handle = '';
    $scope.lowercaseHandle = function(){
        return $filter('lowercase')($scope.handle);
    };
}]);
```

Note how ng-model is used to set up data binding. 

**The Event Loop, Watchers, and the Digest Loop**  
the JS event loop is constantly listening for things like 'keypress', 'click', 'mouseover', 'change', etc. When something happens then it emits an 'event' with the name and other data regarding what occurred - these are typically what jQuery, say, would use as triggers to run some code. Angular adds a whole bunch of other listeners that are contained within the *angular context* and these are known as *watchers*. These watchers keep track of both the old and the new values associated with the watcher - e.g., some text that is ng-modeled to a variable in the scope will have a watcher associated with it. Together all these various watchers are called the watch list. The digest loop/cycle checks all the items in the watch list and compares the old values to the new or current values. If something has changed and is caught by the digest loop, angular then updates everywhere those values are linked or connected - both in the DOM and on the code side. It then runs one more loop to see if anything changed triggers a new event. It continues to do this until all the old and new values are equivalent. This is also called *dirty checking*. This then is the state of our app until the event loop triggers a new digest cycle and the process starts over again.

**Note**: angular is only checking code and events that are happening within the angular context. Consequently, if we use say a regular 'setTimeout()' rather than '$timeout', angular will not automatically catch that change and may fail to update the DOM or run other code accordingly. We can circumvent this issue in various ways:

```javascript
var myApp = angular.module('myApp', []);

myApp.controller('mainController', ['$scope', '$filter', '$timeout', function($scope, $filter, 
    $scope.handle = '';
    $scope.lowercasehandle = function() {
        return $filter('lowercase')($scope.handle);
    };

    // manually watch the scope for changes
    $scope.$watch('handle', function(newValue, oldValue) {
        console.info('Changed!');
        console.log('Old:' + oldValue);
        console.log('New:' + newValue);
    });
        
    // manually tell angular to watch the $scope for changes and trigger a digest cycle
    $timeout(function() {
        $scope.$apply(function() {
            $scope.handle = 'newtwitterhandle';
            console.log('Scope changed!');
        });
        
    }, 3000);
}]);
```

**Common Directives**  

**ng-if="someBooleanExpression"** removes or renders visible a piece of the DOM depending on how the expression evaluates

**ng-show="someBooleanExpression"** and **ng-hide="someBooleanExpression"** shows or hides an element if the expression evaluates to true

**ng-class={ 'someClass': someBooleanExpression}** sets a class when the expression is true. We can add other classes/expressions to evaluate by adding more name/value pairs separated by comas (its just a regular JS object).

**ng-repeat="x in y"** iterates over a list(array) of data creating an element for each.

```html
<div ng-controller="mainController">
            
    <div>
        <label>What is your twitter handle?</label>
        <input type="text" ng-model="handle" />
    </div>
                
    <div class="alert" ng-class="{ 'alert-warning': handle.length < characters, 'alert-danger': handle.length > characters }" ng-if="handle.length !== characters">
                
        <div ng-show="handle.length < characters">
           You have less than 5 characters!
        </div>
        <div ng-show="handle.length > characters">
           You have more than 5 characters!
        </div>
                    
    </div>
                
    <hr />
                
    <h1>twitter.com/{{ lowercasehandle() }}</h1>
                
    <h3>Rules</h3>
    <ul>
        <li ng-repeat="rule in rules">
            {{ rule.rulename }}
        </li>
    </ul>
                
</div>
```

There are a number of other directives - *ng-click, ng-keydown, ng-keyup*, etc - for almost any type of event one can imagine. Check out *ng-cloak* - hides angular placeholders (e.g., '{{ name }}") until the data loads. Bear in mind that one can have multiple directives on the same element (see code above). Check out the API docs to research these directives.

**The XMLHTTPRequest Object**  
just about every modern web framework uses this. Native API built into the browser that allows us to make XMLHttp requests to other urls - originally developed by Microsoft and later adopted by everyone. Can be difficult to use, so most other libraries and frameworks have their own, easier to use, wrappers for it (e.g,, ajax in jQuery). In AngularJS...

**$http**  
this is Angular's much easier to use wrapper for the XMLHttpRequest Object. 

```javascript
var myApp = angular.module('myApp', []);

myApp.controller('mainController', ['$scope', '$filter', function($scope, $filter) {
    
    $scope.handle = '';
    
    $scope.lowercasehandle = function() {
        return $filter('lowercase')($scope.handle);
    };
    
    $scope.characters = 5;
    
    $http.get('/api')
        .success(function(result){
            $scope.rules = result;
        })
        .error(function(data, status){
            console.log(data, status);
        });    
    
    
}]);
```

Check out how to use *$http.post('api', {})* to post data to some api.

## Single Page Applications  

**Multiple Controllers, Multiple Views**  
note that each controller is isolated from any others and has its own $scope variable. 

**Single Page Apps and the '#'**  
the hash is what is known as a *fragment identifier* and has been around html for a while - i.e., it is often used to bookmark certain parts of a page. Interestingly, we do not need to actually link to someplace to be able to access its value via the hash variable on the window object. Angular then uses this to mimic routes/urls to know which code to run... 'http://localhost:3050/api#/bookmark/1', 'http://localhost:3050/api#/bookmark/2', 'http://localhost:3050/api#/bookmark/3', etc..

**Routing, Templates and Controllers**  

index.html

```html
<!DOCTYPE html>
<html lang="en-us" ng-app="myApp">
    <head>
        <title>Learn and Understand AngularJS</title>
        <meta http-equiv="X-UA-Compatible" content="IE=Edge">
        <meta charset="UTF-8">

        <!-- load bootstrap and fontawesome via CDN -->
        <link rel="stylesheet" href="//netdna.bootstrapcdn.com/bootstrap/3.2.0/css/bootstrap.min.css" />
        <style>
            html, body, input, select, textarea
            {
                font-size: 1.05em;
            }
        </style>
        
        <!-- load angular via CDN -->
        <script src="//code.angularjs.org/1.3.0-rc.1/angular.min.js"></script>
        <script src="//code.angularjs.org/1.3.0-rc.1/angular-route.min.js"></script>
        <script src="app.js"></script>
    </head>
    <body>

        <header>
            <nav class="navbar navbar-default">
            <div class="container">
                <div class="navbar-header">
                    <a class="navbar-brand" href="/">AngularJS</a>
                </div>

                <ul class="nav navbar-nav navbar-right">
                    <li><a href="#"><i class="fa fa-home"></i> Home</a></li>
                    <li><a href="#/second"><i></i> Second</a></li>
                </ul>
            </div>
            </nav>
        </header>

        <div class="container">

            <div ng-view></div>
            
        </div>

    </body>
</html>
```

app.js  

```javascript
var myApp = angular.module('myApp', ['ngRoute']);

myApp.config(function ($routeProvider) {
    
    $routeProvider
    
    .when('/', {
        templateUrl: 'pages/main.html',
        controller: 'mainController'
    })
    
    .when('/second', {
        templateUrl: 'pages/second.html',
        controller: 'secondController'
    })

    .when('/second/:num', {
        templateUrl: 'pages/second.html',
        controller: 'secondController'
    });
    
});

myApp.controller('mainController', ['$scope', '$log', function($scope, $log) {
    
    $scope.name = 'Main';
    
}]);

myApp.controller('secondController', ['$scope', '$log', '$routeParams', function($scope, $log, $routParams) {
    
    $scope.name = 'Second';
    $scope.num = $routeParams.num || 0;
    
    
}]);
```

pages/main.html

```html
<h1>Main</h1>
<h3>Scope name value: {{ name }}</h3>
```

pages/seoncd.html

```html
<h1>Second</h1>
<h3>Scope name value: {{ name }}</h3>
<h3>Scope num value: {{ name }}</h3>
```

**Note**: **ngRoute** gives us access to the **$routeProvider** service which lets us specify different hash value patterns, the location of the template we want to use and the controller we want to use on that 'route'. Once all this is set up, angular will then insert that template inside an html element with the *ng-view* attribute/directive attached to it. We can of course use other routers, but angular's ngRoute is sufficient. Take note of how *$routeParams* makes pattern matched urls available in our app. 

**Singletons**  
the one and only copy of an object - angular services are implemented as singletons. We can verify this by appending a property to a service in one controller and then console.logging it out in another controller. Handling services this way saves memory.

*$scope* is an exception. The $scope that we inject into controllers are actually child scopes which are not singletons, but they do inherit from a root scope which is a singleton. 

When we create our own custom services we are also creating singletons.

```javascript
var myApp = angular.module('myApp', ['ngRoute']);

myApp.config(function ($routeProvider) {
    
    $routeProvider
    
    .when('/', {
        templateUrl: 'pages/main.html',
        controller: 'mainController'
    })
    
    .when('/second', {
        templateUrl: 'pages/second.html',
        controller: 'secondController'
    })
    
    .when('/second/:num', {
        templateUrl: 'pages/second.html',
        controller: 'secondController'
    })
    
});

myApp.service('nameService', function() {
   
    var self = this;
    this.name = 'John Doe';
    
    this.namelength = function() {
      
        return self.name.length;
        
    };
    
});

myApp.controller('mainController', ['$scope', '$log', 'nameService', function($scope, $log, nameService) {
    
    $scope.name = nameService.name;
    
    $scope.$watch('name', function() {
        nameService.name = $scope.name;
    });
    
    $log.log(nameService.name);
    $log.log(nameService.namelength());
    
}]);

myApp.controller('secondController', ['$scope', '$log', '$routeParams', 'nameService', function($scope, $log, $routeParams, nameService) {
    
    $scope.num = $routeParams.num || 1;
    
    $scope.name = nameService.name;
    
    $scope.$watch('name', function() {
        nameService.name = $scope.name;
    });
    
}]);
```

Note that we treat our own custom services just like angular's and use dependency injection in the same way. However, depending on how we bind our data, we may need to manually watch ($watch) changes and update our services...

**This way of sharing data across the app is a powerful feature of SPAs. Note, however, that while we can share data within the SPA, a page refresh will cause us to lose all the changed data returning the app to it's initial state. Thus, to ensure a good user experience, we should persist that data either to a cookie, local storage, or perhaps even our DB if there is one.**

Checkout *factories* and *providers* in angular - both very similar to services.

## Custom Directives  

**Variable Names and Normalization**  
to make consistent to a standard - specifically we are dealing with making text strings consistent to a standard. 

**Angular Normalization**  
angular normalizes text in our apps between the models and the views from kebab-case to camel-case - this is necessary since one hyphens represent the subtraction operator in JS. Hence: 
'searchResult' in our JS model or controller would become 'search-result' on the view side of our app and vice versa. 

**Codin a custon directive:**  

app.js

```javascript
myApp.directive("searchResult", function() {
   return {
       restrict: 'AECM',
       template: '<a href="#" class="list-group-item"><h4 class="list-group-item-heading">Doe, John</h4><p class="list-group-item-text">555 Main St., New York, NY 11111</p></a>',
       replace: true
   }
});
```

index.html

```html
<label>Search</label>
<input type="text" value="Doe" />

<h3>Search Results</h3>
<div class="list-group">
    <search-result></search-result>
    <div search-result></div>
    <div class="search-result"></div>
    <!-- directive: search-result -->
</div>
```

**Note** we can use the restrict property in our directive to allow various html elements/attributes to represent our directive:
* 'A' - a custom attribute
* 'E' - a custom element
* 'C' - a custom class
* 'M' - a comment

'AE' is the default and recommended method to use and we do not need to specify anything with a restrict prop on our directive if we are content to use these

**Templates**  
rather than include our template strings in our directives, angular allows us to specify a relative location for a template file:

```javascript
myApp.directive("searchResult", function() {
   return {
       restrict: 'AECM',
       templateURL: 'directives/searchresult.html',
       replace: true
   }
});
```

**Scope**  
be aware that directives inherit the scope from whatever parent element they appear within on the page - so, for example, a directive appearing in a <div> controlled by 'mainController' would have access to and be able to affect any model(s) of that controller. Angular, however, gives us the ability to isolate the scope of any directive by adding a scope prop to our directive as well as the various means to poke holes in this walled garden with:

**Text '@' string value** one-way data binding (*down into the isolate scope*)

```javascript
myApp.directive("searchResult", function() {
   return {
       restrict: 'AECM',
       templateURL: 'directives/searchresult.html',
       replace: true,
       // the 'text' hole
       scope: {
          personName: "@",
          personAddress: "@"
       }
   }
});
```
```html
<h3>Search Results</h3>
<div class="list-group">
    <search-result person-name="{{ person.name }}" person-address="{{ person.address }}"></search-result>
</div>
```

**Object '=' object value** two-way data binding (actually passes the memory location of the object)

```javascript
myApp.directive("searchResult", function() {
   return {
       restrict: 'AECM',
       templateURL: 'directives/searchresult.html',
       replace: true,
       // the 'object' hole
       scope: {
          personObject: '='
       }
   }
});
```
```html
<h3>Search Results</h3>
<div class="list-group">
    <search-result person-object="person"></search-result>
</div>
```

**Note** that the '=' provides two-way data-binding and that we can run into problems with these... consequently when passing objects in this way it is best to try not to affect or mutate the data being passed into the directive from within the directive.

**Function '&' function value**  

```javascript
myApp.directive("searchResult", function() {
   return {
       restrict: 'AECM',
       templateURL: 'directives/searchresult.html',
       replace: true,
       // the 'function' hole
       scope: {
          formattedAddressFunction: '&'
       }
   }
});
```
```html
// template file

<h3>Search Results</h3>
<div class="list-group">
    <search-result person-object="person" formatted-address-function="formattedAddress(aperson)"></search-result>
</div>

// search result directive file
<a href="#" class="list-group-item">
    <h4 class="list-group-item-heading">{{ personObject.name }}</h4>
    <p class="list-group-item-text">
        {{ formattedAddressFunction({ aperson: personObject }) }}
    </p>
</a>
```

**Note** how we have to pass an *object-mapping* to the function
**Also** check out repeating directive with *ng-repeat*

**Compile and Link**  
when building code, the compiler converts the code to a lower-level language, then the *linker* generates a file that the computer will actually interact with. This is not really what angular does... Basically, the *compile function* runs once on the template element before any instance is created and, consequently, before any scope is assigned. The *link function* runs once for each instance once all the instances have been created and a scope created for each - *do not use pre-links as a rule of thumb*, rather use *post-links*. Another way to look at this is that a compile function is used to change the html of the directive itself vs a link function which is used to change some instance of the directive. 

```javascript
myApp.directive("searchResult", function() {
   return {
       restrict: 'AECM',
       templateUrl: 'directives/searchresult.html',
       replace: true,
       scope: {
           personObject: "=",
           formattedAddressFunction: "&"
       },
       compile: function(elem, attrs) {
           
           console.log('Compiling...');
           //elem.removeAttr('class');
           console.log(elem);
           
           return {
               
               post: function(scope, elements, attrs) {
                   
                   console.log('Post-linking...');
                   
                   console.log(scope);
                   
                   if (scope.personObject.name == 'Jane Doe') {
                        elements.removeAttr('class');
                   }
                   
                   console.log(elements);
                   
               }
               
           }
           
       }
   }
});
```

**Note** as it is quite rare to ever need to run compile - we might simply make a new directive in that case - and we are discouraged from use pre-links, angular offers a shorthand to just specify a (post)link like so:

```javascript
myApp.directive("searchResult", function() {
  return {
    restrict: 'AECM',
    templateUrl: 'directives/searchresult.html',
    replace: true,
    scope: {
      personObject: "=",
      formattedAddressFunction: "&"
    },
    link: function(scope, elements, attrs) {
                
      console.log('Linking...');

      console.log(scope);

      if (scope.personObject.name == 'Jane Doe') {
          elements.removeAttr('class');
      }

      console.log(elements);
                
    }           
  }
});
```

**Also** note that one should probably not create super complex link functions for, as they are run for each instance of a directive, when there are many instances (e.g., a long list) the performance of one's app may suffer.

**Transclusion**  
including one document inside of another... place a copy of one document and place it at a specific point inside of another. 

```javascript
myApp.directive("searchResult", function() {
   return {
       restrict: 'AECM',
       templateUrl: 'directives/searchresult.html',
       replace: true,
       scope: {
           personObject: "=",
           formattedAddressFunction: "&"
       },
       transclude: true
   }
});
```
```html
<a href="#" class="list-group-item">
    <h4 class="list-group-item-heading">{{ personObject.name }}</h4>
    <p class="list-group-item-text">
        {{ formattedAddressFunction({ aperson: personObject }) }}
    </p>
    <small ng-transclude></small>
</a>
```
```html
<label>Search</label>
<input type="text" value="Doe" />

<h3>Search Results</h3>
<div class="list-group">
    <search-result person-object="person" formatted-address-function="formattedAddress(aperson)" ng-repeat="person in people">
        *search results may not be valid
    </search-result>
</div>
```

...this basically inserts '*search results may not be valid' in each instance of the directive

## Additional Notes

**ng-submit** basically triggers a submit function on form submission defined in the controller for that form. Allows the user to use the enter key to submit as well as the submit button. As you will probably need to do some redirection on the submit, you will also need to inject the '$location' service into your controller to handle that.

**Note** especially when building larger apps, try to pull as much code as possible out of the controllers and into services focused on a particular task. One injects angular services into one's own custom services the same way as one injects them into controllers (use the minification safe version here as well).

**Nested Controllers, Clean Code and 'Controller As'**

## Wahlin Tutorial - Quick Notes

**ng-cloak** prevents template strings from appearing before the app loads - use at the lowest/smallest possible level (i.e., at a div or whatnot rather than say the body tag).

**Factories and Services**  

**Factories** use to define reusable services or share code and/or state between controllers
* **create and return a custom object**
* created using the *module.factory( )* syntax
* can be injected into other components
* can have its own dependencies
**Services** similar to a factory in terms of functionality
* service function object itself is returned, as opposed to a custom object like a factory - so instead of returning an object with methods and properties attached to it, we just use 'this' and attach them directly to the function itself
* created using the *module.service( )* function
* can also are injected into other components
* can also have its own dependencies 

**Simplified DDO Structure**  

```javascript
angular.module('myModule')
  .directive('myDirective', function() {
    return {
      restrict: 'AECM',
      scope: {},
      template: '<div>Some HTML</div>',  // or a templateURL
      controller: controller,
      link: function(scope, element, attires) {}
    };
  });
```

**'Link' Function**: links the data to the view. The three params above are always passed, in that order, to the link function by Angular - hence we can name them whatever we want. 'scope' will be either the shared or isolate scope of the directive. 'element' will be the element to which the directive is applied. 'attrs' is all the different attributes applied to that tag/element. 

**jqLite**  
is a tiny, API-compatible subset of jQuery that allows Angular to manipulate the DOM. Note: if you load jQuery before you load Angular in your app, Angular will actually replace jqLite with the full jQuery library.

**Some jqLite functions**: Check the docs under 'angular.element' for a reference 
* angular.element( )
* addClass( )/css( )
* attr( )
* on( ) / off( )
* find( ) by tag only
* append( ) / remove( )
* html( ) /  text( )
* ready( )

[**The Difference between compile( ), pre-link( ) and post-link( )**](https://www.jvandemo.com/the-nitty-gritty-of-compile-and-link-functions-inside-angularjs-directives/)