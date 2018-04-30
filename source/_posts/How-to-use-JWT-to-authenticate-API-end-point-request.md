---
title: How to use JWT to authenticate API end-point request
date: 2017-06-24 12:15:31
tag:
  - JavaScript
intro: Introduce the JWT authentication
---

JWT means [JSON Web Token](https://jwt.io/), which is a __compact__ and __self-contained__ authentication method. The main counterpart of JWT might be traditional session or cookies methods which help user keep state and do security authentication check, as the HTTP is stateless. However, from my point, JWT is much more __scalable__, because once server side authenticates and issues a token, the client side has fully control to decide which request will include this token and which are not.

The following image explains the workflow of JWT authentication process:
<!--more-->
![JWT Flow](/images/jwt_auth_flow.png "JWT Flow")

> <i class='fa fa-info-circle'></i> Flow Explain

Take user login process as an example, the client side sends user name and password to server side, once the server sides verified the user information, it will sign a JWT token, and send it back to client side. Then, when next time client side sends request to server side should contain this token, it could be set in the HTTP header, HTTP body, or even URL query string. After that, server side will check this token, if it is correct, server can handle it to next process. Otherwise, 403 will be issued.

Take a TODO list API protection as a real example, more detail will be explained in this example. This example will only take the related code snippet to explain, the full project code could be found here [HiToday](https://github.com/StevenZiu/Hi-Today);

<i class='fa fa-check-circle'></i> __Step One - Setup Secret in config file__

JWT will use a secret to generate a token, as well as verify a token, this secret should be a string. As this string should be constant through the whole App development, we set it in a standalone directory - /config, save as a new file called secret.js

```javascript
  //secret.js
  module.exports = {
    'secret': 'iamstevenzhao'
  };
```

<i class='fa fa-check-circle'></i> __Step Two - Setup Router Handler to Sign a Token__

First in the app.js root server file import this config module then, store this secret in the app object for global access, and also setup a route to handle authentication request.

```javascript
  // app.js
  //import needed modules
  var config = require('./config/secret');
  var authentication = require('./routes/authentication'),
  //make it be global access
  app.set('secret', config.secret);
  //setup authentication api
  app.use('/auth', authentication);
```

The authentication route handler is the place where client side passes the user authentication data, such as the user name, password or email. So here we need to do two things, compare user input with database data, if it is correct, sign a jwt token and send it back to client side, otherwise, send 403 or 400 to client side. In this example, the user input data is stored in the request body. 

The code is here.

```javascript
//authentication.js
var express = require('express');
var authRouter = express.Router();
var jwt = require('jsonwebtoken');

module.exports = authRouter;

// process token included in the post body
authRouter.post('/', function(req, res, next){
  if(req.body.key){
    // store db instance
    var keys = req.keys_db;
    var inputKey = req.body.key;
    // verify the key
    keys.find({key: inputKey}, function(err, data){
      if(data.length){
      // Authenticate successfully
      jwt.sign(data[0].key, req.app.get('secret'), function(err, token){
        if(err) res.status(500).send('Token can not be generated');
        if(token){
          res.status(200).send(token);
          }
        });
      }else{
        // Authentication fail
        res.status(400).send('Password is incorrect');
        }
      });
	}else{
      res.status(400).send('Key can not be empty');
	}
});
```
One thing needs to remind here is that jwt.sign function take the first parameter as the encryption target, and second parameter as secret or key, then a callback function is treated as the third parameter.
More detail about jwt.sign please refer to [here](https://jwt.io/).

In this APP, I use [mongodb](https://www.mongodb.com/) as database, and initializing the database while the app bootstrap, then store the table instance in the req object. As the mongodb is not the main topic of this article, we will talk it in detail in other articles. 

At this moment, we have successfully verify the user indentity, and pass the signed token to client side.

<i class='fa fa-check-circle'></i> __Step Three - Setup frontend js to store the signed token__

How could we store the token in the frontend, there are lots of ways to realize this, it could be stored in the browser, or session, or frontend controller. In this example, frontend logic is controlled by [Angularjs](https://angularjs.org/), and I user a standalone controller to handler auth related staff.

The auth controller is here:

```javascript
// authController
.controller('authController', function($rootScope, $scope, $location, hi, httpCaller){
    var self = this;
    self.authForm = {
        key: ''
    };
    self.submitKey = function(){
      httpCaller
        .auth(self.authForm.key)
        .then(function(token){
          $rootScope.token = token;
          $rootScope.auth = true;
          $location.path('/task');
          }, function(err){
            if(err) hi.callToast(err.data);
          });
    };
})
```
In order to make the app fully modulized, so I also write a standalone service to handle all http request. Once the user click submit button, the submitKey function will be fired, then httpCaller will fire auth function, which will return a promise. If the promise is resolved the token will be sent back, then store it in the $rootScope, which could be accessed through the whole frontend angular logic.

The following is httpCaller service:

```javascript
function httpCaller ($http) {
  var self = this;
  // post to authentication route
  self.auth = function(key){
    return $http({
      method: 'POST',
      url: '/auth',
      data: {key: key}
    });
  };
  //get
  self.getAllTasks = function(token, status) {
    if(status === 'true' || status === 'false'){
      return $http({
        method: 'GET',
        url: '/api/task?token='+token+'&status='+status
      });
    }
    return $http({
      method: 'GET',
      url: '/api/task?token='+token
    });
  };
  //delete
  self.deleteTasks = function(token, parameters) {
    var queryString = '';
    if(parameters.status && parameters.id){
      queryString = '/api/task?status='+parameters.status+'&_id='+parameters.id;
    }else if(parameters.status){
      queryString = '/api/task?status='+parameters.status;
    }else if(parameters.id){
      queryString = '/api/task?_id='+parameters.id;
    }
    queryString += '&token=' + token;

    if(queryString){
      return $http({
        method: 'DELETE',
        url: queryString
      })
    }else{
      return new Error('Delete Task Fail');
    }
  };

  //create
  self.createNewTask = function(token, data){
    if(data !== null && typeof data == 'object'){
      return $http.post('/api/task?token='+token, data);
    }else{
      return new Error('empty task content')
    }
  };

  //update
  self.updateTask = function(token, id, data){
    var queryUrl = "";
    if(id && typeof data === 'object' && typeof data !== 'null'){
      queryUrl = '/api/task?token='+token+'&id='+id;
      return $http.put(queryUrl, data);
    }
  }

}
```

Besides the request auth function, there are other CRUD command functions. Need to notice that, all the CRUD task have included the token in their request, the token could be stored in the request body, or url query string.

Up to now, we have successfully handled the token in the frontend js. The last step will go back to backend, after token included request is sent from the frontend.

<i class='fa fa-check-circle'></i> __Step Four - Setup API protect middleware to verify token__

First we go back to app.js file to setup API route entry point:

```javascript
// app.js
//import api router
api = require('./routes/api');
//CRUD API for Application
app.use('/api/task', api.task);
```

Task router handler will process all the CRUD requests, which is also needed to be protect. That means only the authenticated user can call these API endpoint. So we do it like this:

```javascript
// api.js
/*
 * Serve JSON to our AngularJS client
 */
var express = require('express');
var router = express.Router();
var jwt = require('jsonwebtoken');
var _ = require('lodash');

/**
 * API define here
 * **/

 // varify token
router.use(function(req, res, next){
  if(req.body.token || req.query.token){
    var token = req.body.token || req.query.token;
    jwt.verify(token, req.app.get('secret'), function(err, decoded){
      if(err){
        res.status(403).send('Forbidden! Token is incorrect');
      }
      next();
    });
  }else{
    res.status(403).send('Forbidden! Token missing');
  }
});

router.use(function(req, res, next){
  if(req.task_db){
    next();
  }else{
    next(new Error('No Database Instance'));
  }
});

/**
 * Some general functions for query db
 * **/

function getAll (req, res) {
  req.task_db.find({}, function(err, data){
    if(err) next(new Error(err));
    data = data.reverse();
    res.json(data);
  });
}

/**
 * Preprocess the params from route
 * **/

router.param('status', function(req, res, next, status){
  req.taskStatus = status;
  next();
});

router.param('id', function(req, res, next, id){
  req.taskId = id;
  next();
});

/**
 * url: /api/task
 * method: get
 * behaviour: Display all tasks or filter by status
 * **/

router.get('/', function(req, res, next){
  if(_.isEmpty(req.query)){
    getAll(req, res);
  }else{
    if(req.query.status == 'false' || req.query.status == 'true' ){
      req.task_db.find(req.query, function(err, data){
        if(err) next(new Error(err));
        res.json(data);
      })
    }else{
      getAll(req, res);
    }
  }
});

/**
 * url: /api/task/create
 * method: create
 * behaviour: create a new task, the name can not be duplicate,
 * then fetch all the tasks
 * **/

router.post('/', function(req, res, next){
  if(!_.isEmpty(req.body)){
  //  todo check post data, should have a test function to check req.body data
  //  todo before instance new document
    req.task_db.find({name: req.body.name}, function (err, data) {
      if(!data.length){
        var temp = req.task_db(req.body);
        temp.save(function (err) {
          if(err) next(new Error(err));
          getAll(req, res);
        });
      }else{
        next(new Error("Can not create same task again"));
      }
    });
  }else{
    next(new Error("Create Fail, body is empty"));
  }
});

/**
 * url: /api/task
 * method: delate
 * behaviour: Delete one task by Id or delete all tasks by status
 * if query object is empty, fetch all tasks;
 * **/

router.delete('/', function(req, res, next){
  if(!_.isEmpty(req.query)){
    // delete token before pass it to db query
    delete req.query.token;
    req.task_db.find(req.query, function(err, data){
      if(err) next(new Error(err));
      if(data.length){
        req.task_db.deleteMany(req.query).then(function () {
          getAll(req, res);
        });
      }else{
        next(new Error('Can not find match task to delete'));
      }
    })
  }else{
    getAll(req, res);
  }
});

/**
 * url: /api/task
 * method: update
 * behaviour: update one task
 * if query object is empty, fetch all tasks;
 * **/

router.put('/', function(req, res, next){
  if(!_.isEmpty(req.body) && !_.isEmpty(req.query.id)){
    req.task_db.findByIdAndUpdate(req.query.id, req.body, function(err){
      if(err) next(new Error(err));
      getAll(req, res);
    })
  }else{
    next(new Error('Can not find target to update'));
  }
});

exports.task = router;
```

>__Very important!!!!__
>To make sure all the request should be checked, we should put the token verify middleware at the very top of this router process. Only the token is verified, we call next() to process it to next router handler or middleware function. Otherwise, should redirect to forbidden.

As the jwt.verify function, it takes the token as the first parameter, and secret as the second one, if the token is verified successfully, the err will be empty.

```javascript
// api.js
// token verify
router.use(function(req, res, next){
  if(req.body.token || req.query.token){
    var token = req.body.token || req.query.token;
    jwt.verify(token, req.app.get('secret'), function(err, decoded){
      if(err){
        res.status(403).send('Forbidden! Token is incorrect');
      }
      next();
    });
  }else{
    res.status(403).send('Forbidden! Token missing');
  }
});
```

<i class='fa fa-check-circle'></i> __Conclusion__

This article explains how to implement JSON Web Token to protect API endpoint, as you can see, it's not complicated as we think before. Actually, there are more complicated ways to launch the JWT in the application, for example, you could use some other methods to encrypt token, which will generate a second security layer. Or use more security places to store token. 

Hope this article helps you. Happy Coding.
