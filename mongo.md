MONGODB

## A Quick ES6 Refresher  

**Promises**  
a tool we have for managing asynchronous code throughout or application - i.e., a tool for managing code that we expect to run at some point in the future. A promise will have three possible states:
* *unresolved* waiting for something to finish
* *resolved* something finished and it all went ok
* *rejected* something finished and it went bad  
**Note**: it is really up to you, the developer, what constitutes a resolved or rejected promise.

```javascript
function startGame() {
  let counter = 0;


  document.querySelector('button').addEventListener('click', () => {
    ++counter;
  });


  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (counter > 5) {
        resolve();
      } else {
        reject();
      }
    }, 2000)
  });
}


startGame()
  .then(() => alert('You Win!'))
  .catch(() => alert('Sorry, you lost.'));
```

## Core Fundamentals of MongoDB

Mongoose - ORM / ODM Object Relational Mapper / Object Data Mapper

One can have multiple DBs running inside a single instance of Mongo. (note that one would probably not use multiple DBs within one project and that this feature is more about allowing one to develop across multiple projects).

**_Collections_** - a collection is the core unit of what stores data in a Mongo db. One would typically have a collection for each type of resource one needs to access for their application. CRUD operations will operate on elements (**_documents_**) of a collection - one should rarely if ever mix elements from one collection with those of another.

Note: when initializing a connection to a mongo db, if the db does not exist, mongo will create it on the fly.

A **_Model_** , via a **_Schema_**, represents all the instances of itself that will be saved in a collection. A model is also used directly in the creation of those instances within a specific collection.

Setting up the connection to the mongo database:

```javascript
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost/users_test');

mongoose.connection
  .once('open', () => console.log('Db connection made'))
  .on('error', (error) => {
    console.warn('Warning: ', error);
  });
```

## A Test Driven Experience

Mocha Test Example:

```javascript
describe('Creating records', () => {
  it('saves a user', (done) => {
    const joe = new User({ name: 'Joe' });
    joe.save()
      .then(() => {
        assert(!joe.isNew);
        done();
      });
});   
```

Note: the **_done( )_** function that is available in mocha for handling asynchronous operations.

With mongoose, when a record is created it is automatically assigned and **__id_** by the mongoose ORM, unlike many other ORMs where the id property is not generated until the record is actually saved in the db. However, note that when doing comparisons between _ids, one must call *toString( )* to covert it to a string first! (as the value for the _id is not a primitive).

## Mongoose

**Create**:  
* **_instance.save()_** also checkout findOrCreate()

**Read**:  
* **_class.find([query], [callback])_** finds all documents in the database that match the query. If no query is given, it returns everything.

* **_class.findOne([query], [fields to return], [callback])_** finds one object from the database. If your query matches more than one item in the database, it only returns the first one it finds.
  * If you don't provide a query, it will just return the first Kitten in the database.  
  * If you don't provide a fieldsToReturn, it will return the entire object.  
  * fieldsToReturn can also be in the form of a string with spaces between the field names, e.g. "name age owner", instead of an object.

* **_class.findById([id], [fields to return], [callback])_** Finds a single object in the database by the provided id.  

* **_.where(selector)_** This one is powerful, but a bit more confusing. It allows us to do more complex queries to the database. So far we've done things like "find a Kitten whose name is Dr. Miffles"AND whose color is white AND who is 1 year old. But what if we wanted to query for all Kittens in the Database who were between the ages of 1 and 4? This is when .where( ) comes in. Calling .where( ) on a Mongoose Model (e.g. Kitten) actually returns a Mongoose "Query" object. In order to actually execute the query, we have to call the .exec() method and pass in our usual callback. Example:
  ```javascript
  Kitten.where("age").gte(1).lte(4).exec(function (err, kittens) {  
    // Do stuff
  });
  ```
  What's nice about these queries is that it reads almost like English: "Find all Kittens where the age is greater than or equal to 1 but less than or equal to 4, then execute the following function...".

**Update**  
* **_instance.set( prop: value) and instance.save( )_**  
* **_ instance.update({ prop: value })_**  
* **_class.update({ prop: value }, { prop: newValue })_**  
* **_class.findOneAndUpdate({ prop: value }, {prop: newValue})_**  
* **_class.findByIdAndUpdate(id, { prop: newValue })_**  

**Deleting**  
* **_instance.remove( )_** - removes the instance that calls it  
* **_class.remove( )_** - removes all instances that meet the criteria  
* **_class..findOneAndRemove( )_**  
* **_ class..findByIdAndRemove( )_**  

**Mongo Update Modifiers**  
check out *Update Operators* in the Mongo docs - **_$inc, $mul, etc_**... note that these will be much more performant in that we are just sendng an instruction to the database rather than           retrieving and resaving a record.

**Mongoose-specific Features**  

**Validation**  
* **_.validate()_** - for asynchronous validation  
* **_.validateSync()_** - for synchronous validation  

Sample code for validation in a Schema:

```javascript
const UserSchema = new Schema({
  name: {
    type: String,
    validate: {
      validator: (name) => name.length > 2,
      message: 'Name must be longer than 2 characters.'
    },
    required: [true, 'Name is required.']
  },
  postCount: Number
});
```

**Embedding Resources in Models**  
In a SQL databse, say for a user/blog post db, one would probably have one users table and one posts table, where the foreign key of the posts table referenced the users table. In a NoSQL db, one would still only have a users model, with users and posts schema, and then nest all the posts inside the users records that wrote them  - inasmuch as there would be no post without an author. Posts in an instance of this type would be referred to as sub documents; users, as the primary component of the model, are the documents.  
* note that to add, say a post to the posts array in a user, one retrieves the user record, pushes the post on to the array, and then one must resave the entire record back into the collection.
* to remove a post object from the post array of a user record the process is much the same, except that mongoose has a **_.remove( )_** method one can call one a specific index rather than having to call .splice()).

**Virtual Types**  
Virtuals are fields that don’t exist in the database but act just like normal fields in a Mongoose document. To oversimplify, virtual fields are mock or fake fields that pretend to act like normal ones.
Virtual fields are awesome for creating aggregate fields. For example, if our system requires to have first name, last name and the full name (which is just a concatenation of first two names) – there’s no need to store the full name values in addition to the first and last name values! All we need to do is concatenate the first and last name in a full name virtual. Additionally, virtual types are useful for derivative fields - e.g., for a postCount of post subdocuments.
Another use case is to make the database backward compatible. For example, we might have thousands of user items in a MongoDB collection and we want to start collecting their locations. We have two options: run a migration script to add the default location (“none”) to the thousands of old user documents or use a virtual field and apply defaults at runtime!

To define a virtual we need to:  
1. Call the virtual (name) method to create a virtual type (Mongoose API )
2. Apply a getter function with get (fn Mongoose API)

**Embedded Subdocuments vs Separate Collections**  
Taking a blog post app as an example, note that it could quickly become arbitrarily complex to write queries once we start getting to more complex models - i.e., a Users model where each user has an array of posts/subdocs where each one might in turn have an array of comments/subdocs by different authors, etc.,. As we approach this level of complexity, we ought to start thinking about creating separate collections for users, posts, comments where each collection has some field that references the other collection(s).

Note how this latter structure starts to appear similar to SQL dbs with separate tables for users, posts and comments are connected with foreign keys. The problem with NoSQL dbs/Mongo is that these databases lack the ability to perform single command joins and hence the queries get quite a bit longer as we must pull data from each collection in turn (touch the db multiple times) and this can also cause performance issues.

**Associations**  
See mongoose docs on populating and other  'modifiers' to customize our queries [Mongo Populate](http://mongoosejs.com/docs/populate.html)

**Mongoose Middleware**  

**Pre and Post Hooks**  
these middleware will run either before(pre) or after(post) some specific event.

**_Events_**  
* **_init_** - the initialization of the model  
* **_validate_** - the validation of a record  
* **_save_** - the saving of a record to a collection  
* **_remove_** - the removal of a record from a collection  

Typically we will locate all the middleware associated with a model within that model file

Be wary of cyclical loads - i.e., two files that require each other to function as intended

Example of a *pre remove* middleware:

```javascript
UserSchema.pre('remove', function(next) {
  const BlogPost = mongoose.model('blogPost');  //note how this avoids a cyclical load
  
  BlogPost.remove({ _id: { $in: this.blogPosts } })
    .then(() => next());
});
```

**Skip and Limit**  
* **_skip(n)_** skips n number of records returned
* **_limit(n)_** limits the records returned to n records
* **_sort({n: -1})_** sorts the records returned by a query by prop n  in descending order (1 would be ascending)

**Upstar Music**  
* check out mongo operators - e.g., *$gte, $lte, $text*, etc.  
* note that *$text* searches will try to match a whole string and will not match partial strings
* note that for the $text operator you have to add a index for the field that you want to search and that currently mongo only supports an additional index for only one field (*_id* is indexed by default and does not count against the limit)
* check out *fakerjs*

**Muber**  
* check out supertest for making fake HTTP requests for tests in mocha
* note that mocha, mongoose, node/express don't play well together. Hence, for instance, it is better to access your models in your test suite by requiring the mongoose model rather than requiring the model directly with a require function - i.e.,   
```const Driver = mongoose.model('driver')```  
(if we do it the other way mocha will try to initialize the model again - in addition to the initialization in the app - and we will get a big error message)  
* check out the code for examples of error handling in node/express apps

**Environment Variables**  
**_NODE_ENV_** use this variable to hold 'test' and 'dev' values so that you can tailor logic for your test suite different than your dev environment
* note that **_.findByIdAndUpdate()_** does not return the record that was updated, but rather some statistics about the record that was updated. Consequently, if we then want to assert something related to that record in our test suite for example, we will need to read the record again.

**Geolocations in Mongo**  
Mongo has 1st class support for geo mapping. This works on a lat/lng based system - note however that for Mongo coordinates are lng/lat. One can also use either 2d maps or 2d spheres. Uses GeoJSON objects.
* again note that there are a number of issues working with Mongo and Mocha. See code for examples - parsing query strings into numbers, calling ensureIndex to make sure indices are present ('One Big Gotcha' lecture), etc.,

**CRUD Routes / Methods**  
* Create: POST / create( )
* Read: GET / index( )     where the index method is the read method that returns a number of records or a list
* Update: PUT / edit( )
* Delete: DELETE / delete( )

Checkout [tutorials point](tutorialspoint.com) for straight mongodb tutorial