---
title: Thinking in Document - MongoDB First Bootstrap With Mongoose
date: 2017-06-26 20:46:51
tags: Mongodb
intro: Basic thing about setup MongoDB with Mongoose
---

MongoDB is a NoSQL (not only SQL), or you can call it Object-relational mapping database (ORM), which provides the object-way to manage and manipulate data. There is a bunch of articles on the Internet to do a comparison between MongoDB and MySql, which is not the main concern of this article. The major target of this article is to help MongoDB beginner understand the basic concepts inside the Mongo, and get hands on the mongo. 

This article will contain two parts, the first part will introduce the mongodb structure, second part will give an example about mongo initialising, and how to manipulate data. Some specific discussion of mongo will be published in the other articles.
<!-- more -->
<i class='fa fa-check-circle'></i> __Part One - Collection And Document__

The most important task of DB is to store data, and provide the way for outside to retrieve data. So how are we gonna to store data in mongo way? Let's forget the MySql, take a look at the following image.

![MongoDB Data Structure](/images/mongo-structure.jpg "MongoDB Data Structure")

In the mongodb, we also have a database, but inside the database, there is no table, instead, there are collections. The collections collect one or more document. If it is hard to remember and understand, you can treat the collection as the table, document as the row inside the table. 

But the magic is here, the document is object, which can store anything in any format just like you create the object in Javascript. If one property of the object is complicated, you could even use another object to represent that property, but in mysql, you need to reference other tables to realize this. Besides, collection could have its own methods, for example, you could define a hash function for a collection to hash the password before store it in one document. The amazing things are not only these, but for beginner, switch the way to think in document is the most important. Once we get used to this kind of thinking, we could go deep to understand some much more complex but powerful functions. 

Then, we will take a look at a real example.

<i class='fa fa-check-circle'></i> __Part Two - Example of Mongodb Bootstrap__

In this example, I will do some most basic but important things about how to initialize Mongo, and Create, Read, Update, Delete data inside Mongo. For a better explanation and understanding, I will take contact book as the example data model.

 Some tools will be used here: 

 [Robomongo](https://robomongo.org/) - One GUI for MongoDB
 [Mongoose](http://mongoosejs.com/) - A MongoDB Driver

 First, create a package.json file to include the required dependencies.

 ```javascript
 //package.json
 {
  "name": "mongo-bootstrap",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "dependencies": {
    "mongodb": "2.2.29",
    "mongoose": "4.11.0"
  },
  "author": "stevenzhao",
  "license": "ISC"
}
 ```
Although there is no table, we still need the schema to organize the data inside document. Remember that, the collection collects one or more document, each document could have different schema.

#### Create Database

Before we start everything, you should create a database on the local server (localhost). In the real production development, we seldom use script to create database, because the database will be setup in the local for development, so just go to Robomongo to create a database called __contact-book__.

#### Create Collection

Make a standalone file - contact-book-collection.js, which will define the document schema, and create the collection which contains the document.

```javascript
var mongoose = require('mongoose');
var schema = mongoose.Schema;

// create schema for contack-book document
// remember, each document represents one contact person
var person_schema = new schema({
  name:      {type: String, required: true, maxlength: 20, unique: true},
  number:    {type: Number, required: true, maxlength: 10, unique: true},
  age:       {type: Number, required: false, maxlength: 2},
  createdAt: {type: Date, required: false},
  updateAt:  {type: Date, required: false}
}); 
```

In this example, we can use mongoose Schema function to define schema. For better understanding, we could just think we are defining an object, this object has some properties, and each property will have some requirement which is defined by using object as well. There are many schema types in mongoose, the more detail reference is [here](http://mongoosejs.com/docs/api.html).

After defining the schema, we will implement it to create document and the collection where the document stays.

```javascript
// define the collection with document which implements document schema
// collection contains many documents
var Persons = mongoose.model('person', person_schema, 'persons');

//export this collection
module.exports = Persons;
```

The mongoose.model function takes first parameter as the document name, the second parameter as the document schema, the third will be collection name, if it is ignored, the mongoose will generate one automatically. Then we export this collection.

Mongoose also provides pre save function, which means the function which will be executed before the document saving. For example, hash some sensitive data before save it to database.

```javascript
// define a pre-save function to update date automatically
person_schema.pre('save', function(next){
  var current_time = new Date();
  this.updateAt = current_time;
  if(this.createdAt === undefined){
    this.createdAt = current_time;
  }
  // call next pre-save function or other functions
  next();
});
```

In this example, we define a pre save function to generate the created date and updated date.

Now, we have successfully defined the document schema, and use it to create a collection. The full file is here:

```javascript
//contact-book-collection.js
var mongoose = require('mongoose');
var schema = mongoose.Schema;

// create schema for contack-book document
// remember, each document represents one contact person
var person_schema = new schema({
  name:      {type: String, required: true, maxlength: 20, unique: true},
  number:    {type: Number, required: true, maxlength: 10, unique: true},
  age:       {type: Number, required: false, maxlength: 2},
  createdAt: {type: Date, required: false},
  updateAt:  {type: Date, required: false}
}); 

// define a pre-save function to update date automatically
person_schema.pre('save', function(next){
  var current_time = new Date();
  this.updateAt = current_time;
  if(this.createdAt === undefined){
    this.createdAt = current_time;
  }
  // call next pre-save function or other functions
  next();
});

// define the collection with document which implements document schema
// collection contains many documents
var Persons = mongoose.model('person', person_schema, 'persons');

//export this collection
module.exports = Persons;
```

#### Create Database Connection and Insert Data

Once created the collection, it's time to put the collection in the database. Create another file called mongo-bootstrap.js. First we need to import the collection, then connect the database.

```javascript
var mongoose = require('mongoose');
// import Persons collection
var Persons = require('./contact-book-collection');
// initialize the connection
mongoose.connect('mongodb://localhost/contact-book');
// store the connection instance
var connect = mongoose.connection;
```

After we import the collection definition, we have not connected it with any database, so we connect the local database __contact-book__ which should be created manually in the Robomongo ni the first step. Then, we store the connection instance to a variable, which will be used to listen database connection status. Once the database connects successfully, we could insert data.

```javascript
// watch the database events
connect.on('error', function(err){
  console.log(err);   
}); 

connect.on('open', function(){
  console.log('database connect successfully!');
})
```

Now we have successfully connected the database __contack-book__, also import the collection definition. It's time for us to insert some data.

Remember that, for mongo, we use document to represent data, and document actually is object, so we could create document just like we create object in javascript.

```javascript
// create document object
var person = new Persons({
  name: 'steven',
  number: 68183333,
  age: 25
});
```

Now, we have created a person document. In order to make the person is unique in the collection, so we need to check whether there is a duplicate record in collection, so we call __find__ function to check inside collection, there will be more examples explain how to query data later. If no duplicate record, call __save__ function to insert this document. __save__ will also take a callback function.


```javascript
// first way to initilize one document
Persons.find({name: person.name, number: person.number}, function(err, data){
  if(err) return false;
  if(data.length > 0){
    // the document already in the collection
    console.log('data already in the collection');
    return false;
  }else{
    person.save(function(err){
      if(err){
        console.log(err);
      }else{
        console.log('data insert successfully');
      }
    })
  }
});
```

We also can create new document directly from collection.

```javascript
// second way to initilize one document
Persons.create({
  name: 'Jack',
  number: 68182333,
  age: 25
}, function(err){
  if(err){
    console.log(err);
    return false;
  }else{
    console.log('data insert successfully');
  }
})
```

or Multiple inserting by using array.

```javascript
var multipleInsert = 
  [{name: 'Jane Doe', number: 89792312, age: 13},
  {name: 'Jane Doe 1', number: 89792322, age: 23},
  {name: 'Jane Doe 2', number: 89792342, age: 33},
  {name: 'Jane Doe 3', number: 89792362, age: 43}];

Persons.create(multipleInsert, function(err){
  if(err) {
    console.log(err);
    return false;
  }else{
    console.log("multiple insert successfully");
  }
})
```

__The second parameter for create function is a callback function, if we do not pass this function to create, this query will not execute, instead, a query string will be returned, then we will need to call exec() function manually. This feature enables a way to chain multiple query together to finish some much more complicated query__

You will see some examples following.

The full file is here:

```javascript
var mongoose = require('mongoose');
// import Persons collection
var Persons = require('./contact-book-collection');
// initialize the connection
mongoose.connect('mongodb://localhost/contact-book');
// store the connection instance
var connect = mongoose.connection;
// watch the database events
connect.on('error', function(err){
  console.log(err);   
}); 
connect.on('open', function(){
  console.log('database connect successfully!');
  // insert data
  var person = new Persons({
    name: 'steven',
    number: 68183333,
    age: 25
  });
  // first way to initilize one document
  Persons.find({name: person.name, number: person.number}, function(err, data){
    if(err) return false;
    if(data.length > 0){
      // the document already in the collection
      console.log('data already in the collection');
      return false;
    }else{
      person.save(function(err){
        if(err){
          console.log(err);
        }else{
          console.log('data insert successfully');
        }
      })
    }
  });
  // second way to initilize one document
  Persons.create({
    name: 'Jack',
    number: 68182333,
    age: 25
  }, function(err){
    if(err){
      console.log(err);
      return false;
    }else{
      console.log('data insert successfully');
    }
  })
  var multipleInsert = 
    [{name: 'Jane Doe', number: 89792312, age: 13},
    {name: 'Jane Doe 1', number: 89792322, age: 23},
    {name: 'Jane Doe 2', number: 89792342, age: 33},
    {name: 'Jane Doe 3', number: 89792362, age: 43}];
  Persons.create(multipleInsert, function(err){
    if(err) {
      console.log(err);
      return false;
    }else{
      console.log("multiple insert successfully");
    }
  })
})
```
#### Data GET, UPDATE, DELETE

Let's create a new file to explain how to do data get, update and delete.

Same as create the data, we first need to connect the database, and import the collection. Remember that, the document is stored inside the collection.

```javascript
var mongoose = require('mongoose');
//connect db
mongoose.connect('mongodb://localhost/contact-book');
// import collection
var Persons = require('./contact-book-collection');
// store the connection instance
var connection = mongoose.connection;
``` 

All the query in the mongo is asynchronous, which is realised by using promise, which provides two ways to finish query, one is callback function, the other is promise chain. If you we do not pass the callback function, the query will not be executed, only return it back. In order to execute it we need to call exec() function manually. Before we call exec(), we could chain more queries to execute much more complicated query function, which is benefited from promise chain.

Simple GET:

```javascript
connection.on('open', function(){
  // find all (callback)
  Persons.find({}, function(err, data){
    if(err) console.log(err);
    if(data){
      console.log('Find All');
      console.log(data);
    }else{
      console.log('No match query');
    }
  });
  //find all (query chain)
  var query = Persons.find({});
  query.exec(function(err, data){
    if(err) console.log(err);
    if(data){
      console.log('Find All (query chain)');
      console.log(data);
    }else{
      console.log('No match query');
    }
  });
});
```

First find all uses callback way, second one uses promise way, it only return a query string, the query will not be executed until we call exec(), and exec() will also take a callback function. The result of them is same.

The following query first gets the document, then gets the property of that document. If you are familiar with MySql, should be easy to understand select.

```javascript
// find one with select property
Persons.findOne({name: 'steven'}, 'age number', function(err, data){
  if(err) console.log(err);
  if(data){
    console.log('Find one with select property');
    console.log(data);
  }else{
    console.log('No match query');
  }
});
// find one with select property (query chain)
var query = Persons.findOne({name: 'steven'});
query.select('age number');
query.exec(function(err, data){
  if(err) console.log(err);
  if(data){
    console.log('Find one with select property (query chain)');
    console.log(data);
  }else{
    console.log('No match query');
  }
})
```

Thanks to the promise, we could chain query together to complete much more complicated GET query:

```javascript
// complex query (query chain)
Persons
  .find({})
  .limit(2)
  .where('age').gt(20).lt(30)
  .select('name number')
  .exec(function(err, data){
    if(err) console.log(err);
    if(data){
      console.log('Complex Query');
      console.log(data);
    }else{
      console.log('No match query');
    }
  });
```

UPDATE:

Same as GET, we also have two ways to execute UPDATE query.

```javascript
//update (query chain)
Persons
  .where({name: 'steven'})
  .update({number: 99999999})
  .exec(function(err){
    if(err){
      console.log(err);
    }else{
      console.log('Update Data Successfully (query chain)');
    }
  });
//callback way is missing here, just need to put the callback as the second parameter of update function
```

Then REMOVE:

```javascript
// remove (callback)  
Persons.remove({name: 'Jane Doe'}, function(err){
  if(err) {
    console.log(err);
  }else{
    console.log('remove data successfully');
  }
});
```

Actually, the mongoose provides many powerfull functions to execute query, such as findOneAndRemove, findOneAndUpdate, or deleteOne, deleteMany, etc.
But for beginner, the most important thing is to switch the way to think about data storing. Not Only SQL, the object-related data structure can help build much more complicated but easy understanding data structure. 