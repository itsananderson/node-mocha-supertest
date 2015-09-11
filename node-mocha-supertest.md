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
===

* http://mochajs.org/
* Simple syntax
* Many test supported reporters

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

Testing responses with a custom validator function:

```javascript
supertest("/nav")
    .expect(200)
    .expect(function(res) {
        assert(res.body.prev, "Expected prev link");
        assert(res.body.next, "Expected next link");
    })
    .end(done);

```

---

Lots more validation examples:

https://github.com/visionmedia/supertest#api

---

Testing external APIs

```javascript
var supertest = require("supertest")("http://www.google.com/");

// http://google.com/foo
supertest.get("/foo")
    .end();

// http://google.com/search?q=test
supertest.get("/search?q=test")
    .end();
```

---

Other supertest examples


```javascript
// Uploading a file
supertest
    .post("/upload")
    .attach("fieldName", "path/to/file.jpg")
    .end();
```

---

Other supertest examples

```javascript
// Set a header and send a JSON body
supertest
    .post("/foo")
    .set("Content-Type", "application/json")
    .send({
        message: "Hello, Server!"
    })
    .end();
```

---

Inversion of Control
===

* Components don't construct their own dependencies
* Decouples your application
* Drastically simplifies testing

---

Before IoC:

```javascript
function UserManager() {
    this.db = new DBConnection("username", "password");
    this.hasher = PasswordHashers.SHA256;
}

UserManager.prototype.validateLogin = function(username, password) {
    var user = this.db.users.findByUsername(username);
    return user && user.password === this.hasher(password);
};

// ...
```

---

After IoC:

```javascript
function UserManager(dbConnection, hasher) {
    this.db = dbConnection;
    this.hasher = hasher;
}

// ...
```

---

Testing a component with IoC:

```javascript
var UserManager = require("../user-manager");

function fakeHasher(input) {
    return output + "5";
}

var fakeDbConnection = {
    users: {
        findByUsername: function() {
            {
                username: "foo",
                password: fakeHasher("1234")
            }
        }
    }
};

var manager = new UserManager(fakeDbConnection, fakeHasher);

describe("UserManager", function() {
    it("validates a username and password", function() {
        assert(manager.validateLogin("foo", "1234"));
    });
});
```

---

Composing Express middleware:

```javascript
var users = require("./routes/users");
var friends = require("./routes/friends");
var messages = require("./routes/messages");
var auth = require("./routes/auth");

var app = require("express")();

app.use(auth.router());
app.use(users);
app.use(auth.authenticate(), friends);
app.use(auth.authenticate(), messages);

app.listen(process.env.PORT || 3000);
```

---

Testing Express middleware:

```javascript
var app = require("express")();
var messages = require("../routes/messages");

app.use(function(req, res, next) {
    req.user = {
        id: 1
    };
    next();
});

app.use(messages); // No authentication

var supertest = require("supertest")(app);


// ... mocha tests
```

---

Sinon.JS
===

* Stubs
* Spies
* Mocks

---

## Sinon Stubs

## http://sinonjs.org/docs/#stubs

---

```javascript
// Create a single function
var returnTrue = sinon.stub().returns(true);
var returnFalse = sinon.stub().returns(false);
var throwsError = sinon.stub().throws(new Error("some error"));
```

---

```javascript
// Return different values for different inputs
var factorial = sinon.stub();
factorial.withArgs(1).returns(1);
factorial.withArgs(2).returns(2);
factorial.withArgs(3).returns(6);
```

---

```javascript
// Modify an object
var stubbed = sinon.stub(dbConnection.users, "findByUsername").returns({
    username: "foo",
    passwrod: hash("1234")
});

stubbed.restore(); // Restore original method
```

---

## Sinon Spies

## http://sinonjs.org/docs/#spies

---

Spy on an anonymous function

```javascript
var callback = sinon.spy();
PubSub.subscribe("message", callback);
PubSub.publishSync("message");
assert(callback.called);
```

---

Spy on an existing function

```javascript
var spyHash = sinon.spy(hash);
var manager = new UserManager(db, spyHash);

manager.validateLogin("foo", "1234"); // Should call hashing function

assert(spyHash.called);
```
