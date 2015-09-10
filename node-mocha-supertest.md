title: Testing Node APIs With Mocha and SuperTest
author:
    email: will@itsananderson.com
    name: Will Anderson
    twitter: itsananderson
    github: itsananderson
output: output/index.html
controls: true

---

Will Anderson  
will@itsananderson.com  
http://willi.am/  
https://github.com/itsananderson  
https://twitter.com/itsananderson  

---


Where to find these slides:

http://willi.am/node-mocha-supertest

---

#### So you want to write a Node API?

---

#### So you want to write a Node API?

#### Great! How are you going to test it?

---

Mocha

http://mochajs.org/

---

A Simple Example:

```javascript
var someModule = require("../");
var assert = require("assert");

describe("Some module", function() {
    it("does some thing", function() {
        assert(someModule.doesSomeThing());
    });
});
```

![](http://itsananderson.blob.core.windows.net/post-images/simple-mocha-test.png)

---

Basic Mocha components:

`describe` - High level grouping (suite) of tests  
`it` - A single test function. Usually tests one feature or edge case

Advanced Mocha components:

`before` - Run once before all tests in a test suite  
`after` - Run once after all tests in a test suite

`beforeEach` - Run before every test in a test suite  
`afterEach` - Run after every test in a test suite

---

A note on async code:

```javascript
it('can do async work', function(done) {
    setTimeout(function() {
        done();
    }, 1000);
});
```

---

More advanced example:

```javascript
var app = require("../");

describe("Advanced module", function() {
    var server;
    var port;
    before(function(done) {
        server = app.listen(0, done);
        port = server.address.port();
    });
    
    after(function() {
        server.stop();
    });
    
    // other tests here
});
```

---

Supertest
===

* Takes care of server setup and teardown
* Simplifies request and response validation
* Makes tests very easy to read

---

Before supertest

```javascript
var app = reqiure("../");
var request = require("request");
var assert = require("assert");

it("Responds with 'Hello, World!'", function(done) {
    request("http://localhost:" + port + "/", function(response, body) {
        assert.equal(response.statusCode, 200);
        assert.equal(body, "Hello, World!");
        done();
    });
});
```

---

After supertest:

```javascript
var app = require("../");
var supertest = require("supertest")(app);

it("Responds with 'Hello, World!'", function(done) {
    supertest("/")
        .expect(200)
        .expect("Hello, World!")
        .end(done);
});
```

---

Testing JSON responses:

```javascript
var app = require("../");
var supertest = require("supertest")(app);

it("Responds with 'Hello, World!' object", function(done) {
    supertest("/json")
        .expect(200)
        .expect({
            message: "Hello, World!"
        })
        .end(done);
});
```

---

Testing responses with a RegEx:

```javascript
var app = require("../");
var supertest = require("supertest")(app);

it("Responds with 'Hello, World!' object", function(done) {
    supertest("/")
        .expect(200)
        .expect(/Hello.*/)
        .end(done);
});
```

---

Lots more examples:

https://github.com/visionmedia/supertest#api

---

Testing external APIs

```javascript
var supertest = require("supertest")("http://google.com/");

supertest.get("/foo") // Sends a request to http://google.com/foo
    .end();
```

---

Other examples


```javascript
// Uploading a file
supertest
    .post("/upload")
    .attach("fieldName", "path/to/file.jpg")
    .end();
```

---

Other examples

```javascript
// Set a header and send a JSON body
supertest
    .post("/foo")
    .set("Content-Type", "application/json")
    .send({message: "Hello, Server!")
    .end();
```
