# SectServer
## An easy to use, fast and simple web framework for node.js with built-in template engine.

`npm i sectserver`

```javascript
const sectserver = require("sectserver")

var app = sectserver();

app.set('static dir', __dirname + '/public');
app.set('views dir', __dirname + '/views');

app.add({
  url: '/',
  handler: (req, res) => {
    res.send('Hello, World!');
  }
})

app.listen(3000);
```

# Example:

```javascript
const sectserver = require("sectserver")

var app = sectserver();

app.set('static dir', __dirname + '/public');
app.set('views dir', __dirname + '/views');

app.add({
  url: '/',
  handler: (req, res) => { // > /
    res.render('Hello {globe}, My Name Is: {cool_name}!', {globe: `World`, cool_name: `SectServer`}); // Default Render Engine: SectPlating, See Guide To Change The Default.
  }
})

app.add({ // :one = required, :one? = optional
  url: '/test/:one?', // > /test/world?two=SectServer
  handler: (req, res) => {
    res.render('Hello {one}, My Name Is: {two}!', {one: `${req.params.one}`, two: `${req.query.two}`});
  }
})

app.add({
  url: '/body', // > /body
  handler: (req, res) => {
    res.json(`${JSON.stringify(req.body)}`);
  }
})

app.add({
  url: '/hi', // > /hi
  handler: (req, res) => {
    res.send('Hello!');
  }
})

app.add({
  url: '/end', // > /end
  handler: (req, res) => {
    res.end('The End!');
  }
})

app.listen(8080);
```

# Guide:

After you install the module, require it:
```javascript
const sectserver = require('sectserver');
```
initialize your app:
```javascript
let app = sectserver();
```

If you want, change the default configs:
```javascript
app.set('template', 'pug') /* view engine, see: https://www.npmjs.com/package/consolidate#supported-template-engines for a list of supported view engines | view engine defaults to: sectplating */
.set('static dir', './public') /* static content directory (.css, .js, .json...)*/
.set('views dir', './views'); /* views directory ( .pug, .haml, .html) */
app.locals.foo = 'bar'; /* app.locals is an object that you can use (and call) it everywhere (middlewares, routers, renders...)*/
```
Now, you can add the middlewares you want
```javascript
app.add(compress()) /* data compress*/
.add(favicon('./public/favicon.ico')) /* serve favicon and cache it*/
.add(app.serveStatic('./public')) /* serve static content */
.add(bodyParser.json()) /* data parser to req.body */
.add(bodyParser.urlencoded({ extended: true })) /* same above */
.add(cookieParser()) /* cookies parser to req.cookies */
```
you can set routers for a path (or all)  and a method through the 'app.add' method.

| Param | Object |
|-----|---------|
| req | Request. |
| res | Response. |
| next | Next middleware to call. |

```javascript
app.add(
    (req,res,next) => {
        res.render("index", { title: 'My Title Page'});
    }
);
```
##Response
instance of http.ServerResponse.
```javascript
res.send('foo'); /* => send 'foo' */
res.status(404); // response status is 404
res.status(404).send('Not Found'); /* => send error 404 and 'Not Found' */
res.sendStatus(404); /* => same above */
res.json({'foo': 'bar'}) /* => send '{'foo':'bar'}'*/
res.sendFile('/test.json') /* => send the content of file /public/test.json (or your static dir)*/
res.render('index',{foo: 'bar',bar: 'foo'}) /* => send the view index.pug (default, or your views engine)*/
res.redirect('/foo') /* => redirect users to /foo */
res.locals /* => is similar to app.locals but only lives in current request (you can refresh it inn each request through middlewares) */
```

##Router
Its a handler for your paths. You can to nest routers on the app.
```javascript
let router = app.Router();

router.add({
        url: '/path', // > /path
        method: 'GET',
        handler: (req,res,next) => { /* anything */}
    })
    .add({
        url: '/path', // > /path
        method: 'POST',
        handler: (req,res,next) => { /* anything */}
    })
    .add({
        url: '/path', // > /path
        method: 'PUT',
        handler: (req,res,next) => { /* anything */}
    })
    .add({
        url: '/path', // > /path
        method: 'DELETE',
        handler: (req,res,next) => { /* anything */}
    })
    .add({
        url: '/path', // > /path
        method: 'PUT',
        handler: (req,res,next) => { /* anything */}
    })
    .add({
        url: '/user/:id', // > /user/48
        handler: (req,res,next) => { /* req.params.id */}
    })
    .add({
        url: '/asset/', // > /asset?name=logo.png&from=images
        handler: (req,res,next) => { /* req.query.name, req.query.from */}
    })

/* incorpore to your app */
app.add({
    url: '/foo',
    handler: router
    })
```
