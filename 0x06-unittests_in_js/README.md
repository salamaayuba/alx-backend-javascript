How to Test Node.js Apps using Mocha, Chai, and SinonJS
Draft updated on Invalid Date
Node.js
Default avatar
By Joy Warugu

How to Test Node.js Apps using Mocha, Chai, and SinonJS
This tutorial is out of date and no longer maintained.

Introduction
Not having tests in your app is a pain because chances are every time you make slight adjustments to your app you have to manually check every single part of your app to see if anything broke. Writing tests, however, also feels for the most part a chore. But we definitely need them.

Why the fuss around automated testing? With automated tests you don’t have to test all the workings of the app every time you change something in your enormous and still growing app. This means that when you are adding or modifying parts of your app, you will have a lot more confidence in the reliability of your code. Which is really reassuring.

In this article, we’ll look into testing node apps. We’ll use Mocha, Chai, and SinonJS, and delve into using spies, stubs, and mocks. You’ll need to be familiar with Node.js and JavaScript, to get the most out of this. Enjoy!

Literature on Tools
Mocha
Mocha is a test runner. This just means that it is a tool that runs and executes our tests. The tests themselves aren’t written in Mocha. Other test runners include Jasmine, Jest.

Let’s install mocha globally to be able to use it on our command line:

npm install -g mocha
Create our new directory for our app:

mkdir testing-async-code && cd testing-async-code
npm init
npm install --save-dev mocha
mkdir tests
Then add a basic test just to see how Mocha works. For now, we’ll use Assert as our assertion library which comes in Node.js so no npm install is needed.

const assert = require("assert");

describe("smoke test", function() {
  it("checks equality", function() {
    assert.equal(true, true);
  });
});
To run this we just run the command mocha and pass in the location of our tests:

mocha tests/
Yay, our test passes. What essentially happens is that the assert tests something, if the test passes it doesn’t do anything; if it fails it throws an exception and this stops your tests. Just to see how a failure would look like we can change the line to something that would fail.

const assert = require("assert");

describe("smoke test", function() {
  it("checks equality", function() {
    assert.equal(true, false);
  });
});
Notice our tests stop and we get an error message telling us we have an AssertionError and what is expected. Assert, throws an error and that is how Mocha knows whether our tests are failing or passing.

For the entire article, we won’t use arrow functions. Why? You ask: Well, with Mocha you can access its context using this keyword. So, for example, if you wanted to extend the timeout for a test which is normally 2000ms you could do something like: this.timeout(5000). When using arrow functions, it makes this bound to the lexical context, not the Mocha context. This is all to say that doing something like this.timeout(5000) won’t work while using the ES6 arrow functions, along with all the other stuff you can do with this in Mocha. You can read more on this here.

Chai
Chai is an assertion library. So far, we have just used Assert to make assertions and we have an idea of what asserts do. Chai, does the exact same thing but allows for better readability.

Let’s install it:

npm install --save-dev chai
Now we can change our smoke test to read like this:

const chai = require("chai");
const expect = chai.expect;

describe("smoke test", function() {
  it("checks equality", function() {
    expect(true).to.be.true;
  });
});
SinonJS
SinonJS provides stand-alone test spies, stubs, and mocks. This is the mechanism we’ll be using to create our spies, stubs, and mocks.

npm install --save-dev sinon
Spies: Creates fake functions which we can use to track executions. This means we can determine whether the function has been executed, how many times its been called, etc. We can also use spies on existing functions and get the same capability, to track those functions’ executions. We’ll see this in action a bit later.

Stubs: Enables us to replace functions. This gives us more control. We can return whatever we want or have our functions work in a way that suites us to be able to test multiple scenarios.

Mocks: They are fake methods, that have pre-programmed behavior and pre-programmed expectations.

Spies, Stubs, Mocks
We have a basic understanding of what these are. Let’s try them out. Create a new file on your project folder; /controllers/app.controller.js.

Spies
Since we are doing Node testing, its safe to assume that we’ll have a function for our endpoints that will look something like this:

module.exports = {
  // A func that takes in two parameters `req` and `res` [request, response]
  getIndexPage: (req, res) => {
    res.send("Hey");
  }
}
While testing this we require a function that takes in two parameters for our tests. Create a new file inside the tests folder; /tests/controllers/app.controller.test.js

We can just instantiate req and res to empty objects and use that.

const chai = require("chai");
const expect = chai.expect;
// import our getIndexPage function
const indexPage = require("../../controllers/app.controller.js");

describe("getIndexPage", function() {
  it("should return index page", function() {
    let req = {}
    // Have `res` have a send key with a function value because we use `res.send()` in our func
    let res = {
      send: function() {}
    }

    indexPage.getIndexPage(req, res)
  });
});
This doesn’t really give us anything to test or much control over the function. We could use a spy and then make some assertions on the spy. Making assertions on a spy is possible because the spy gives us a placeholder function that we can use to track our function’s execution.

const chai = require("chai");
const expect = chai.expect;
// import sinon
const sinon = require("sinon");
const indexPage = require("../../controllers/app.controller.js");

describe("getIndexPage", function() {
  it("should return index page", function() {
    let req = {}
    // Have `res` have a send key with a function value because we use `res.send()` in our func
    let res = {
      send: sinon.spy()
    }

    indexPage.getIndexPage(req, res);
    // let's see what we get on res.send
    console.log(res.send);
  });
});
Now, when we run our tests, which we’ll do by running mocha tests/**/*.* (we have to tell mocha where our tests live); we can see what res.send returns. It’s basically a list of all the methods we could call to make assertions for our tests.

Let’s go ahead and make some of these assertions:

We’ll expect that our function res.send is called once
We’ll expect that we get an argument Hey on the firstCall
const chai = require("chai");
const expect = chai.expect;
// import sinon
const sinon = require("sinon");
const indexPage = require("../../controllers/app.controller.js");

describe("getIndexPage", function() {
  it("should send hey", function() {
    let req = {}
    // Have `res` have a send key with a function value because we use `res.send()` in our func
    let res = {
      // replace empty function with a spy
      send: sinon.spy()
    }

    indexPage.getIndexPage(req, res);
    // let's see what we get on res.send
    console.log(res.send);
    // `res.send` called once
    expect(res.send.calledOnce).to.be.true;
    // expect to get argument `Error` on first call
    expect(res.send.firstCall.args[0]).to.equal("Error");
  });
});
When we run this, notice we get a failing test. This is because we expect to get an argument Error instead of Hey. We can go back to our code and replace Error with Hey and run our tests again. This way, they pass. I put this in here because sometimes it’s worthwhile to make your tests fail first so that you make certain that you aren’t experiencing false positives. From here on feel free to fail the next tests on purpose and then make them pass, if you haven’t been doing that already.

What if we had an already existing function and we just want to spy on that function? We can simulate this scenario by defining a simple function like so:

const chai = require("chai");
const expect = chai.expect;
// import sinon
const sinon = require("sinon");
const indexPage = require("../../controllers/app.controller.js");

const user = {
  addUser: (name) => {
    this.name = name;
  }
}

describe("AppController", function()  {
  describe("getIndexPage", function() {
    it("should send hey", function() {
      let req = {}
      // Have `res` have a send key with a function value because we use `res.send()` in our func
      let res = {
        // replace empty function with a spy
        send: sinon.spy()
      }

      indexPage.getIndexPage(req, res);
      // let's see what we get on res.send
      // console.log(res.send);
      // `res.send` called once
      expect(res.send.calledOnce).to.be.true;
      expect(res.send.firstCall.args[0]).to.equal("Hey");
    });
  });
});
describe("User", function() {
  describe("addUser", function() {
    it("should add a user", function() {
      sinon.spy(user, "addUser");

      // lets log `addUser` and see what we get
      console.log(user.addUser);
    });
  });
});
From our console.log you can see, we get all the watch functionality on our already existing function. We can go ahead and make some assertions on this function just like we did before.

const chai = require("chai");
const expect = chai.expect;
// import sinon
const sinon = require("sinon");
const indexPage = require("../../controllers/app.controller.js");

const user = {
  addUser: (name) => {
    this.name = name;
  }
}

...

describe("User", function() {
  describe("addUser", function() {
    it("should add a user", function() {
      sinon.spy(user, "addUser");
      user.addUser('John Doe');

      // lets log `addUser` and see what we get
      console.log(user.addUser);
      expect(user.addUser.calledOnce).to.be.true;
    });
  });
});
So far, we know how to use spies; both to create dummy functions and to wrap already existing functions so that we are able to watch and test our functions. You’ll find that most of the time we’ll probably not use spies but they are pretty useful to test whether callbacks have been called and how many times they were called during our code execution. Spies are the simplest to use in SinonJS and most functionality is built on top of them. They are a pretty good starting point with SinonJS.

Stubs
Stubs are really great. This is because; they have all the functionality of spies but unlike spies, they replace the whole function. This means that with spies the function runs as is but with a stub you are replacing said function. This helps in scenarios where we need to test:

External calls which make tests slow and difficult to write (e.g., HTTP calls and DB calls)
Triggering different outcomes for a piece of code (e.g., what happens if an error is thrown and if it passes)
Enough talk let’s dig into stubs:

We’ll add a function to our app controller. Just to broaden the scope of things to test. In our /controllers/app.controller.js we can add this code.

module.exports = {
  // A func that takes in two parameters `req` and `res` [request, response]
  getIndexPage: (req, res) => {
    if (req.user.isLoggedIn()) {
      return res.send("Hey");
    }
    res.send("Ooops. You need to log in to access this page");
  }
}
Now, in our getIndexPage function we check to see if a user isLoggedIn to send the message Hey otherwise we send another message, Ooops. You need to log in to access this page.

isLoggedIn in this case is a function that does some checks to see whether the current user is logged in. It could be that it checks the users’ JWT tokens or make direct calls to the database, we really don’t have to know what the function does exactly. We’ll just imagine it’s some expensive operation happening that either returns true when the user is logged in or false otherwise.

Let’s test it. In our test file we can replace what we had with this:

const chai = require("chai");
const expect = chai.expect;
// import sinon
const sinon = require("sinon");
const indexPage = require("../../controllers/app.controller.js");

describe("AppController", function() {
  describe("getIndexPage", function() {
    it("should send hey when user is logged in", function() {
      // instantiate a user object with an empty isLoggedIn function
      let user = {
        isLoggedIn: function(){}
      }

      // Stub isLoggedIn function and make it return true always
      const isLoggedInStub = sinon.stub(user, "isLoggedIn").returns(true);

      // pass user into the req object
      let req = {
        user: user
      }

      // Have `res` have a send key with a function value because we use `res.send()` in our func
      let res = {
        // replace empty function with a spy
        send: sinon.spy()
      }

      indexPage.getIndexPage(req, res);
      // let's see what we get on res.send
      // console.log(res.send);
      // `res.send` called once
      expect(res.send.calledOnce).to.be.true;
      expect(res.send.firstCall.args[0]).to.equal("Hey");

      // assert that the stub is logged in at least once
      expect(isLoggedInStub.calledOnce).to.be.true;
    });
  });
});
What we did is stub out the isLoggedIn function and made it return true always for this case. The stub replaces isLoggedIn and now we can test scenarios where the user is logged in. In our case when the user is logged in we expect to see the message Hey. If we run our tests now mocha tests/**/*.*, they should pass. We can also now test when the user is logged out by making our stub return false.

...

describe("AppController", function() {
  describe("getIndexPage", function() {
    it("should send hey when user is logged in", function() {
      ...
    });

    it("should send something else when user is NOT logged in", function() {
      // instantiate a user object with an empty isLoggedIn function
      let user = {
        isLoggedIn: function(){}
      }

      // Stub isLoggedIn function and make it return false always
      const isLoggedInStub = sinon.stub(user, "isLoggedIn").returns(false);

      // pass user into the req object
      let req = {
        user: user
      }

      // Have `res` have a send key with a function value because we use `res.send()` in our func
      let res = {
        // replace empty function with a spy
        send: sinon.spy()
      }

      indexPage.getIndexPage(req, res);
      // let's see what we get on res.send
      // console.log(res.send);
      // `res.send` called once
      expect(res.send.calledOnce).to.be.true;
      expect(res.send.firstCall.args[0]).to.equal("Ooops. You need to log in to access this page");

      // assert that the stub is logged in at least once
      expect(isLoggedInStub.calledOnce).to.be.true;
    })
  });
});
That’s how stubs work in a nutshell. You can go crazy with them and throw specific errors and such. Check them out on the docs. Mocks is up next!!
