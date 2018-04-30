---
title: News Feed App by Vue.js
date: 2018-01-15 12:06:14
intro: Let's build a news feed App with Vue
tag:
  - Vue
  - JavaScript
---

In this article, we talk about how to use vue.js 2.x to build a simple News Feed App. 

About [vue.js](https://vuejs.org/). There is a bundle of articles on the Internet talking about the advantages of Vue.js, it combines the good parts from Angular and React, and removes the notorious parts from them. If you want to know more about the vue.js can check its [magic document](https://vuejs.org/).

The target app in this article is a news feed app, which can get the latest break news from 60 main news publishers around the world. The source code of the app could be found [here](https://github.com/StevenZiu/newsfeed).
<!--more-->
This project will go through some main concepts of Vue, including component, router, and vue app structure. In order to fast construct a workable vue app structure, we use [vue-cli](https://github.com/vuejs/vue-cli) to bootstrap the app, which provides basic webpack config and module hot load function. The webpack will be another topic which will not be discussed in this article. We also use [News API](https://newsapi.org/) to get news content and headlines.

Let's move on!

#### Bootstrap App Structure. 

First, we need to have a minimum workable vue app, which will be done by using official vue command line tool vue-cli, which could be installed by npm or brew.

```javascript
//Prerequisites: Node.js (>=4.x, 6.x preferred), npm version 3+ and Git.
npm install -g vue-cli;
//$ vue init <template-name> <project-name>
vue init webpack news_today;
```

This command will generate a basic app structure. Then go to this project folder - news_today.

```javascript
// install dependencies
npm install;
// start the app local server
npm run dev;
```

Then app page will open automatically. As following:

![Starting Page](/images/news-feed/1.png "Starting Page")

And we have a well-rounded code structure. The src directory contains the whole vue app. Inside the src, the components directory includes all the vue components. Router directory contains the router definition. Need to notice that, vue requires a root app and a main js file as the app entry point.

```javascript
// main.js
// The Vue build version to load with the `import` command
// (runtime-only or standalone) has been set in webpack.base.conf with an alias.
import Vue from 'vue'
import App from './App'
import router from './router'
import VueResource from 'vue-resource'
import VueMaterial from 'vue-material'

Vue.config.productionTip = false;

Vue.use(VueResource);
Vue.use(VueMaterial);

/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: { App }
})
```
In the main.js, we just initialize a vue root app, everything starts from here. After we create a vue root instance, it points to a root vue component. And includes the router definition in the root vue instance. Besides, we also use vue-material as our frontend style library, vue-resource as our HTTP client.

```html
<template>
  <div id="app">
    <router-view></router-view>
  </div>
</template>

<script>
export default {
  name: 'app',
}
</script>
<style src="vue-material/dist/vue-material.css"></style>
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```
As the App vue component, in the template tag, we bind our vue app to it by using 'id=app', then provide a router-view tag which will be filled by using other vue components according to the current url path.

__Each vue component contains three parts, which are the template, script, and style, a template is the view of the vue component, the script includes the data logic of this component, and provides the endpoint for outside accessing, the style is the style for this component or global style. And we export this component by calling export default in the script, which makes this component could be reusable.__

Now let's create our own app.

#### Setup App Router

In the main.js, we have imported the router, and bind it with our root app, which defined our vue app router logic.

```javascript
// router/index.js
import Vue from 'vue'
import Router from 'vue-router'

// import the components
import Hello from '@/components/Hello'
import All from '@/components/All'
import News from '@/components/News'
import Publisher from '@/components/Publisher'

Vue.use(Router);

export default new Router({
  routes: [
    {
      // Homepage
      path: '/',
      name: 'Hello',
      component: Hello
    },
    {
      // Publisher List
      path: '/publisher',
      name: 'Publisher',
      component: Publisher
    },
    {
      // Display selected publisher news
      path: '/news/:id',
      name: 'News',
      component: News
    }
  ]
});

```

If you are familiar with angularjs, you should be familiar with this one. First, we import all the components which will be used in the different routers. '@/**' will help vue know this is a vue component. Besides, we also import vue-router and vue. Vue-router is vue official router library, you can use other third parties router library. Call vue.use() will load depending libraries. Look like express, right? XD

Then we also need to call export to expose this router. Routes are stored in an array, each route element contains the path, and an optional name then is the component used in this path. Still remember router-view? Yes, the component in the route element will be put inside the router-view tag, which makes the SPA (single page application).

In this application, we have three routes, hello points to the homepage, publisher route will list all the publishers. Then a news route will parameter id points the news feed page of some specific publisher.

We have created our router file. Then it's time to add components.

#### Create Components

We will have four main components in this vue app. Let's write them one by one.

__Homepage Component - Hello__

```html
<template>
  <div class="hello">
    <h1>Vuejs Project - News Feed</h1>
    <!--<router-link :to="{ name: 'All'}">All News Feed</router-link>-->
    <router-link :to="{ name: 'Publisher'}">Publishers List</router-link>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data () {
    return {

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>

```

The Hello component is quite simple, which just needs to provide a link to get all the publishers list. 

```html
<router-link :to="{ name: 'Publisher'}">Publishers List</router-link>
```

Need to notice that, router-link could be treated as <a> tag in the general html. ':' equals to 'v-bind', 'to' is a directive points to the target link when user clicks this link. Because we have assigned the name to each route when we define them in the router/index.js, so we just pass the route name here is enough. 

In the script, except the name, we also have a data function, which returns the data scope of this component, that means this returned data object can be accessed only inside this component. As the reason why it must be a function can refer [here](https://vuejs.org/v2/api/#Options-Data).

The following is our completed hello page.

![Hello Page](/images/news-feed/2.png "Hello Page")


__Publisher Component__

When user clicks Publisher List, it will go to Publisher component.

```html
<template>
  <md-layout>
    <md-layout md-row>
      <md-layout v-for="publish in publishers" :key="publish.id" md-flex="33" md-align="center">
        <md-card md-with-hover class="card">
          <md-card-header>
            <div class="md-title">{{publish.name}}</div>
          </md-card-header>

          <md-card-content>
            {{publish.description}}
          </md-card-content>

          <md-card-actions>
            <md-button @click="readMore(publish)">About this publisher</md-button>
            <md-button @click="getFeed(publish)">Get News From</md-button>
          </md-card-actions>
        </md-card>
      </md-layout>
    </md-layout>
  </md-layout>
</template>

<script>
  export default{
    data() {
      return {
        publishers: '',
      }
    },
    methods: {
      initData: function() {
        this
          .$http
          .get('https://newsapi.org/v1/sources?language=en')
          .then(function(res){
            console.log(res);
            this.publishers = res.body.sources;
          })
          .catch(function(err){
            this.publishers = err;
          })
      },
      readMore: function(publish){
        window.open(publish.url, '_blank');
      },
      getFeed: function (publish) {
        this.$router.push({name:'News', params: {id: publish.id}})
      }
    },
    created: function(){
      this.initData();
    }
  }
</script>

<style>
.card{
  width: 80%;
  margin-bottom: 20px;
}
</style>
```

First, let's go inside the script tag. In the data, we just initialize an empty string to store all the publishers. The methods are some customized function to be used inside this component. The initData is the function we get the publisher list from the API, then store the data to publishers, 'this' refers to this component, '$http' is the service provided by vue-resource. 

Here is the API call example:

![API Publisher Get](/images/news-feed/5.png "API Publisher Get")


When data of publishers changed from an empty string to an array, vue will automatically refresh the data in the template. We also have a readMore function to open the publisher website, and a getFeed function to get the news content from specific news publisher. 

When user clicks one publisher to see news content of that publisher, we need to redirect the user to our 'News' route with a publisher id. Vue provides a programmatic way to realize this.

```javascript
this.$router.push({name:'News', params: {id: publish.id}})
```

this.$router will refer to current component router information, call push function will build a route stack, means user can easily go back by poping up this route.

Besides the data, methods, we also have a created property, which means the function will be fired when the component created, just like jquery document.onready. In this example, we will call our initData function when a component is created.

Now we have prepared the publisher data when this component is created, it's time to render it in the template tag. __A very important concept is that, the vue template can only access the context of this vue component. Than means, the template can only access data or methods inside the script of this vue component. It can not access the parent or child component's context unless we pass them to this component.__ This concept helps us keep the data safe and prevents the context pollution.

In the template we use [v-for](https://vuejs.org/v2/guide/list.html#v-for) to render each element of publishers array. v-for is just like ng-repeat in the angular. Then we bind the unique id to the key, which is necessary to avoid duplicate, more explain about the key is [here](https://vuejs.org/v2/guide/list.html#key).

Then we render each publisher as a [card](http://vuematerial.io/#/components/card), in vue we also can use "{{publish.name}}" to interpolate the data, which is similar with handler-bar, or angular. Then we bind two click events to each card.

```html
<md-card-actions>
  <md-button @click="readMore(publish)">About this publisher</md-button>
  <md-button @click="getFeed(publish)">Get News From</md-button>
</md-card-actions>
```

If you want to pass parameters to click function, no need to interpolate it, because the parameters have already known by vue if we use v-on or '@' to listen to events.

Now, when user fires getFeed function, it will hit the path '/news' with publishing id as parameter. Then the router/index.js will help redirect to /news and render News component.

The following is our publisher list page:

![Publisher Page](/images/news-feed/3.png "Publisher Page")

It's time to construct News component.

__News Component__

The first thing we need to do inside News component is to get the publisher id from url. The vue provides a native object to get current route status, it could be accessed by calling this.$route, so let's get publisher id by using this way.

Inside the script created function.

```javascript
created: function () {
  this.publishId = this.$route.params.id;
  this.initData(this.publishId);
}
```

Then we need to fire the news API to get the news content from this publisher. So we call a initData function to realize this.

```javascript
methods: {
  initData: function (id) {
    let queryString  = '';
    queryString = "https://newsapi.org/v1/articles?source="+id+"&apiKey=aa7f04f1086f4454bc2041b677f3fd26";
    this
      .$http
      .get(queryString)
      .then(function(res){
        console.log(res.body.articles);
        this.feedNews = res.body.articles;
      })
      .catch(function(err){
        this.feedNews = err;
      })
  }
}
```

In the initData function we first combine a query string, then pass it to api endpoint.

Here is news API example:

![News API Get](/images/news-feed/6.png "News API Get")

 After data is back, store the data in the feedNews variable.

```javascript
data(){
  return{
    publishId: '',
    feedNews: ''
  }
}
```

That's what we need to do in the script tag, now we need to render the feedNews array. We will create a standalone news card component to render each news of feedNews array, so here we just need call v-for to pass news to news card.

The full file is here:

```html
<!--News.vue-->
<template>
  <div>
    <md-layout md-gutter md-row md-align="start center">
      <card v-for="news in feedNews" :news="news"></card>
    </md-layout>
  </div>
</template>

<script type="text/javascript">
  import Card from '@/components/card.vue'
  export default {
    name: 'News',
    data(){
      return{
        publishId: '',
        feedNews: ''
      }
    },
    methods: {
      initData: function (id) {
        let queryString  = '';
        queryString = "https://newsapi.org/v1/articles?source="+id+"&apiKey=aa7f04f1086f4454bc2041b677f3fd26";
        this
          .$http
          .get(queryString)
          .then(function(res){
            console.log(res.body.articles);
            this.feedNews = res.body.articles;
          })
          .catch(function(err){
            this.feedNews = err;
          })
      }
    },
    components: {
      Card,
    },
    created: function () {
      this.publishId = this.$route.params.id;
      this.initData(this.publishId);
    }
  }
</script>
```

Other things need to note: in the script, we also have a component object which refers to the other components we will use here. In this example is Card component. Then in the template:

```html
<card v-for="news in feedNews" :news="news"></card>
```

__We have bound each news in feedNews to a property called "news". Maybe a little bit confused here, just think that we need a tunnel to connect the parent component (News component) and child component (Card component), that's property. The child component will provide some properties to be used by parent components to pass some data inside.__ In this example, card component will provide a property called "news".

In the Card component script:

```javascript
<script type="text/javascript">
  export default {
    name: 'Card',
    props: ['news'],
    methods: {
      readMore: function(){
        window.open(this.news.url, '_blank')
      }
    }
  }
</script>
```

Now in the Card component template, we could directly interpolate some needed information from news property.

Full Card.vue is here:

```html
<template>
  <md-layout md-flex="33" md-row md-align="center">
    <md-card md-with-hover>
      <md-card-header>
        <div class="md-title">{{news.title}}</div>
        <div class="md-subhead">{{news.author}}</div>
      </md-card-header>

      <md-card-media>
        <img :src="news.urlToImage">
      </md-card-media>

      <md-card-content>
        {{news.description}}
      </md-card-content>

      <md-card-actions>
        <md-button @click="readMore">Read More</md-button>
      </md-card-actions>
    </md-card>
  </md-layout>
</template>

<script type="text/javascript">
  export default {
    name: 'Card',
    props: ['news'],
    methods: {
      readMore: function(){
        window.open(this.news.url, '_blank')
      }
    }
  }
</script>
<style>
  .md-theme-default.md-card {
    background-color: #fff;
    width: 70%;
    margin-bottom: 20px;
  }
</style>
```

Need to note: property could be defined with some validation check, refer [here](https://vuejs.org/v2/guide/components.html#Prop-Validation).

News feed page:

![News Feed Page](/images/news-feed/4.png "News Feed Page")

#### Conclusion

That's it, we have built a very basic news feed app by using vue. Compared with Angular, vue needs more thinking from component side, everything is component in the vue if you want. And we don't need to config too much before we start the project, which makes it simpler than Angular. 

Hope you enjoy this sharing, there will be more articles about Vue in the future.

Happy Coding!