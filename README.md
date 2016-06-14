# Cottage

[![Join the chat at https://gitter.im/therne/cottage](https://badges.gitter.im/therne/cottage.svg)](https://gitter.im/therne/cottage?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
Cottage is the [Express](http://expressjs.com) style lightweight API routing framework on [koa.js](https://github.com/koajs/koa).<br>

```
$ npm install --save cottage
```

### Features
- No callback hells. of course it's on the top of Koa.js!
- Express style routing.
- No `res.send`, `this.body`. Just `return` the response. Like [Sinatra](http://www.sinatrarb.com)
- Pre-definable Response (`Status`)

### Preview
```js
var cottage = require('cottage');
var app = cottage();

app.get('/hello', function*(req) {
    return 'Hello world!';
});

app.get('/user/:id', function*(req) {
    var user = yield User.get(req.params.id);
    return user;
});

app.all(function*(req) {
    return 404;
});

// start server
var koa = require('koa');
koa().use(app).listen(8080);
```

#### Status
Status is basically HTTP status code / message pair, but you can predefine the Status and get it by name.
```js
var cottage = require('cottage');
var Status = cottage.Status;
var app = cottage();

app.get('/users/me', function*(req) {
    // Send HTTP Status Code / Message pair.
    if (!req.header.Token) return Status(401, 'Unauthorized');
    // response body will be: {"msg":"Unauthorized"}
    
    // or you can use predefined one...
    var user = yield User.auth(req.header.Token);
    if (!user) return Status('authFailed');
});

// predefine the Status responses
Status.predefine({
    'authFailed': { status: 401, code: 2000, msg: 'Auth Failed.' },
});

// or...
Status.predefine('notFound', { status: 404, code: 1000, msg: 'Not Found' });
```

#### Express-like Routing
You can do almost the whole things that you've did on the Express.

**NOTE THAT** Few things differ from express:
- `req` is [`koa.Request`](http://koajs.com/#request)
- `res` is [`koa.Response`](http://koajs.com/#response)
- `next` is Generator that points next middlewares
- `this` is [`koa.Context`](http://koajs.com/#context)


```js
var router = cottage();

router.get('/', function*(req, res, next) {
    // this = koa.Context, req = koa.Request, res = koa.Response, next = Generator
    return 'Hello';
});

// write middleware in Koa.js style
router.use(function*(next) {
    console.log('Middle man ' + this.request.ip);
    yield next;
});

// param callback
router.param('userId', function*(req, res, next, userId) {  
    req.user = yield User.auth(userId);
    yield next;
});

// other routing style...
router.route('/:userId')
    .get(getUser)
    .put(editUser)
    .del(removeUser);

// finally... nested routing!
var app = cottage();
app.use('/user', router);

// attach to koa. fits well with koa middlewares
var koa = require('koa');
koa()
    .use(function* benchmark(next) {
        var start = new Date;
        yield next;
        console.log('Took %d ms', new Date - start);
    })
    .use(app)
    .listen(8080);

```
