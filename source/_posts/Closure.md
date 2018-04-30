---
title: Discuss Closure
date: 2018-04-28 15:19:41
intro: Talk about Closure with examples and questions
comment: true
---

When talk about JavaScript, we must talk __closure__ which is both dangerous and powerful native attribute of JavaScript. We may have already used it everyday, just not pay attention to it. When confusedness comes out, we observe it. 

In this brief article, I will share my own understanding of the basic concept of __closure__ with example explanations, also list a few common closure interview questions at the end.
<!--more-->
More detail explain can refer [here](https://github.com/getify/You-Dont-Know-JS/blob/master/scope%20%26%20closures/ch5.md)

I hope this article can help some closure beginners to pick it up quickly before diving deep.

### What is closure?

By definition:
 >closure is when a function is able to remember and access its lexical scope even when that function is executing outside its  lexical scope.

__Lexical scope__ just means the code scope during the author time. When I write the following code, it will natively have the lexical scope.
```javascript
function foo () {
  // lexical scope start
  var a = 2
  function bar () {
    console.log(a)
  }
  bar()
  // lexical scope end
}
```
In the function *foo*, it creates a protected scope inside it function body, which actually is the lexical scope of the function *bar*. So __closure__ means when function *bar* is fired in the outside of function *foo*, it will get the access to original scope, it knows the variable a equals 2 in this case.

To be specifically, the lexical scope of *bar* is not only the body of function *foo*, the outside of the *foo* is also the lexical scope of *bar*. But for better understanding, we limit the lexical scope to the area where has variable definition.

The definition is not complicate, but the real situation might go confuse, let's see some examples

### Get hands dirty

Example I:
```javascript
function foo () {
  // lexical scope start
  var a = 2
  function bar () {
    console.log(a)
  }
  return bar
  // lexical scope end
}
var baz = foo ()
baz() // output: 2
```
In the example I, after the execute of *foo*, the return has been assigned to baz, at this moment, JavaScript will clean the memory of function *foo* to save more clean space. But when we fire *baz*, the magic appears, it still remembers that the varible a equals to 2. Because of __closure__, the *bar* will still remember the lexical scope when it was authoring.

Example II:
```javascript
var fn
function foo () {
  var a = 2
  function bar () {
    console.log(a)
  }
  fn = bar
}
function baz () {
  fn ()
}
foo ()
baz () // output: 2
```
In the Examlpe II, we do not return *bar*, instead, we assign it to a globle variable fn, which will be called when we fire funciton *baz*. Similarly, after function *foo* is fired, the body of *foo* will be cleaned, means the assignment of *a* will not be here anymore, but function *bar* still get the reference copy of its lexical scope when it was authoring. That's why the output still is 2.

Example III:
```javascript
for (i = 0; i < 3; i++) {
  // for block start
  setTimeout(function () {
    console.log(i)
  }, 1000)
  // for block end
}
```
Things get complicate, what's the output of the Example III? The answer is *3 3 3*.
Why they are three 3? I guess most of closure begineers will think the result is *0 1 2*.

First I want to explain why the result is 3. In JavaScript, there is a timeout queque, when timeout is setup inside the for loop, each loop will push a new timeout to queue, when for loop is finished, the timeout queue will be fired together. So the end status of *i* is 3, that's the reason why it's 3.

Another question is that, why all the results are same?

In the for block, all the function will share the same reference of *i*, that means each loop will not have a new scope instance, they are all in the same scope. Therefore, in this example, the final status of *i* is 3, so when for loop ends, there will be three 3.

If we want to make the result as our expectation, we can take advantage of *closure*. According to the *closure* definition, the function can remember its lexical scope in the authoring time. In the above example, the parent lexical scope of timeout callback function is the scope of for loop block. So they have the same reference to *i*. We just need to make callback function has its own new lexical scope in the each for loop.

Let's try *IIFE* first:

Example IV:
```javascript
for (i = 0; i < 3; i++) {
  // for block start
  (function () {
    setTimeout(function () {
      console.log(i)
    }, 1000)
  })()
  // for block end
}
```

IIFE will generate a standalone scope, but in this example we do nothing, the *i* will still have direct refernce to *i* in the for loop. Let's pass *i* as a function parameter.

Example V:
```javascript
for (i = 0; i < 3; i++) {
  // for block start
  (function (i) {
    setTimeout(function () {
      console.log(i)
    }, 1000)
  })(i)
  // for block end
}
```
Yup! We got what we expected now! The result of this function will be *0 1 2*.

The difference between this example with previous one is that, we created a closured scope, and this scope will have different *i* in the each iteration. When callback function is fired, although it is out the original scope, it still get the access to the *i* we passed to the closured scope.

Similarly way, we can use *let* keyword to create a new scope in the each iteration.

Example VI:
```javascript
for (i = 0; i < 3; i++) {
  // for block start
  let j = i
  setTimeout(function () {
    console.log(j)
  }, 1000)
  // for block end
}
```

*Let* keyword is introduced in the ES6, which will make variable be scoped. So in the each iteration, there is a new closured scope which includes the different *i*. Then, when the callback is fired, it can get the access to that scope when it is authored.

### Summary

*Closure* is a native character of JavaScript, means the function can keep the access to its lexical scope even when this funciton is executed outside the original scope. So if we want to take advantage of closure, we need to create a scope which includes both the function and the states that function wants to remember.

### Common Questions

1. Write a function which allow you to do this:
```javascript
var addSix = createBase(6);
addSix(10); // returns 16
addSix(21); // returns 27
```
2. Consider the following code. What will be printed on the console if a user clicks the first and the fourth button in the list? Why?
```javascript
var nodes = document.getElementsByTagName('button');
for (var i = 0; i < nodes.length; i++) {
   nodes[i].addEventListener('click', function() {
      console.log('You clicked element #' + i);
   });
}
```
3. Fix the previous question’s issue so that the handler prints 0 for the first button in the list, 1 for the second, and so on.
