frex.js Web Framework
=======

A framework for creating modern web applications, it's easy-use for front-end developer and JavaScript people. frex.js is NOT a traditional MVC web framework. It aims to provide a new way to develop web service without server-side knowledge.

With integration of front-end/back-end web development experiences, it uses pure JavaScript method instead of tranditional web communication methods. There is no need to understand GET/POST methods of HTTP protocol and implement AJAX stuffs directly, developer totally can write a web service with front-end web development experence only.

frex.js is based on Express web framework in Node.js, it means developer who is already familiar with node.js development do not need to learn new things for using frex.js, and furthermore, it can support all express middleware. If you do not want to use special feature of frex.js, you can still do your job under old school MVC model.

Features
-

* MVC (Model–view–controller)
* Easy-use route manager (based on [courser](https://github.com/cfsghost/courser))
* JavaScript RPC
* Access control for RPC
* Native EventEmitter integration for RPC
* Support real-time purpose
* Support all middlewares of Express web framework
* Support cookie-based session

Installation
-

Install directly via NPM

    npm install frex.js

Getting started
-

frex.js is based on Express web framework, you can just write your web app with traditional way.

___`<project directory>`/app.js:___
```js
var Frex = require('frex.js');

var app = Frex();

app.get('/', function(req, res){
        res.send('Hello World');
});

app.listen(8080);
```

***

### Managing Routes with frex.js

frex.js is using "routes" directory for putting route rules by default, it supports all features of [courser](https://github.com/cfsghost/courser).

___`<project directory>`/routes/index.js:___
```js
module.exports = {
        '/': function(req, res) {
                res.render('index');
        }
};
```

***

### New Way to Make Your Service APIs with Engine

Engine is the fast way to implement your own service APIs, using JavaScript RPC for communicating between client and server. Futhermore, having pure javascript experience is enough for it without understanding HTTP methods, AJAX and polling things.

You should put all engine files in the "engine" directory.

___`<project directory>`/engine/myengine.js:___
```js
var MyEngine = function() {
    this.message = 'Hello world';
};

MyEngine.prototype.sum = function(a, b, callback) {
    callback(a + b);
};

module.exports = {
        type: 'engine',
        engine_name: 'MyEngine',
        prototype: MyEngine
};
```

The Engine will be running on server-side, but it's magic that you can call functions of "MyEngine" and get/set value from variable of backend engine instance on client-side. 

```html
<script type="text/javascript" src="/frex"></script>
<script>

    // Pass your params if you need.
    App.setConnectionParams({
        "_csrf": "here is your CSRF token."
    });

    App.require('MyEngine', function() {
        var myEngine = App.Engine('MyEngine');

        // Get value to variable of backend engine
        console.log(myEngine.message);

        // Call function of backend engine
        myEngine.sum(1, 3, function(result) {
            console.log(result);
        });

        // Set value to variable of backend engine
        myEngine.message = 'Hello Engine';
    });
    
</script>
```

### Calling APIs of Engine on the Server-side

Calling APIs of Engine on client-side is not only way to use engine, you can call it on the server-side as well.

Here is example to render web page on the server-side with calling APIs of engine:
```js
module.exports = {
    '/': function(req, res) {
        var myEngine = req.frex.Engine('MyEngine');
        myEngine.sum(1, 3, function(result) {

            res.render('index', { result: result });
        });
    }
};
```

***

### Protect the APIs of Engine with Session

In fact, we hope some of APIs are not available for anybody, it only can be accessed by user who has permission.

Here is example to make access control for engine with Session:
```js
module.exports = {
        type: 'engine',
        engine_name: 'MyEngine',
        prototype: MyEngine,
        check_permission: function(data, callback) {
            if (!data.req.session.login)
                callback(false);
            else
                callback(true);
        }
};
```

***

### Using Event Emitter to Hook Engine for Realtime Purpose

It's possible to hook class of engine with event emitter. With event emitter support, you can make real-time service easily.

__Server-side__
```js
var util = require('util');
var events = require('events');

var MyEngine = function() {
    var self = this;
    self.count = 0;

    // Fire event per second
    setTimeout(function() {
        self.count++;
        self.emit('pump', self.count);
    }, 1000);
};

util.inherits(MyEngine, events.EventEmitter);
```

__Client-side__
```js
myEngine.on('pump', function(count) {
    console.log(count);
});
```
* You can see more details from `examples/chat'

### Using CSRF middleware with Engine

Pass your own params ( like CSRF token ) to the Engine. Using `setConnectionParams` method before you require your Engine.

__Client-side__
```js
App.setConnectionParams({
    "_csrf": '/* your CSRF token here. */'
});

App.require('MyEngine', function() {
    /* code here */
});
```
* You can see more details from `examples/csrf'

License
-
Licensed under the MIT License

Authors
-
Copyright(c) 2013 Fred Chien <<cfsghost@gmail.com>>
