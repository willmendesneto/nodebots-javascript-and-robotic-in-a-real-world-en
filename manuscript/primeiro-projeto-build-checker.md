# First project: Build Checker

## What is a pipeline build

In some presentations I realized that this term is not known
for many people, even those who are more familiar with approaches
such as continuous integration and continuous delivery.

Build pipeline is a concept that was built in the middle of 2005
and it's based on the idea of task parallelization, separating
each step into small acceptance criteria for the application. It
is worth remembering that these steps can be automatic or manual.


## Creating a Build Checker

Whenever we get in touch with the Arduino, for example, we make
the example of flashing the LED's, commonly known as blink.

In this example I will show you a more attractive way of
approaching this example for our everyday life, based on models
such as [Hubot](https://hubot.github.com/) and [Retaliation](https://github.com/codedance/Retaliation) to check our build pipeline and
find out the health of our application using Arduino + NodeJS +
Johnny-Five in an introduction to NodeBots.


## Anatomy of a build checker

The project was based on [CCmenu](http://ccmenu.org/), a project
created by ThoughtWorker Erik Doernenburg to check and show the
status of a particular project on a seamless integration server.

In our case we get the idea for something physical, using open
hardware and NodeJS. Our application will consume an XML with
the information returned by Travis-CI. From this data we check
the current state of the application and return it using some
artifices like Arduino and LEDs to warn our team that something
wrong happened with our build and we must correct as soon as
possible.


## Materials needed

For this project we will use:

- 1 Protoboard: A protoboard is nothing more than a plate with holes and conductive connections for mounting experimental electrical circuits, without the need for welding. A simple protoboard costs between $ 5.00 to $ 10.00 and can be found in any electric store;
- 2 LEDs (*light-emitting diode*): 1 red to signal the broken build and 1 green to signal that the build was successfully completed. An LED costs less than $ 0.50 and can be found at any electrical store;
- Arduino with 2 GND (ground) ports;

![Material needed for Build Checker](images/image47.png)

GND ports are called "ground" ports. They are electrical conductors that connect to the Earth - that is, to the Electric Earth. Since it is always neutral and (theoretically) present in every electric circuit, it is always taken as reference point for the measurement of potentials, containing zero volts.

Now just plug the 2 LEDS, linking as follows:

- Successful build on the number 12 door + a GND port of our Arduino;
- Error build on port number 10 + a GND port of our Arduino;

The following image illustrates the assembly of the components with the Arduino.

![Connecting the Arduino: ports and LEDs](images/image12.png)


### Controlling the LED

Knowing the components that we will use, we now go to our initial code that will control our LED. First we will create the folder `src`, where will be the code of our application.

Let's create the folder for our `build-checker` project and navigate to the created folder.

```bash
$ mkdir build-checker
$ cd build-checker
```

We will start our application by typing the `npm init` command with the` -y` flag which means that all the answers that were previously made will be answered and registered with the default response. After this we will install the johnny-five package locally as a dependency in the folder of our project.

```bash
$ npm init -y
$ npm install --save johnny-five
```

Inside the folder of our project we will create the folder `src` and inside it our file` index.js`, where will be our content.

```bash
$ mkdir src
$ touch src/index.js
```

In the `index.js` file we will import the Johnny-five package using the` require` command and create our protoboard instance to add the LED.

```javascript
...
var five = require('johnny-five');
var board = new five.Board();
...
```

For the first activity with the component we will use a new Johnny Five class called `LED`. To use this class we need to pass the value of the pin that the `LED` is connected to the Arduino.

```javascript
...
var led = new five.Led(12);
...
```

Our first functional code will look something like this.

```javascript
var five = require('johnny-five');
var board = new five.Board();

board.on('ready', function() {
  var ledSuccess = new five.Led(12);
  var ledError = new five.Led(10);

  ledSuccess.blink();
  ledError.blink();
});
```

Now let's put our code to work on our Arduino. Inside the folder of our project, we will enter in our command prompt/terminal the following command:

```bash
$ node src/index.js
```

After this command our prompt/command line is showing that the command has run successfully and the result will be that our two LEDs will be flashing.

Quite simple, is not it? In the next topic we will think a little more about our architecture.


## Creating the CI/CD build information request

We already have our build checker abstraction, let's now add the last functionality that is to read the build info on our continuous delivery server and/or continuous integration server.

From there we created a request to read the content via GET in [SNAP-CI](https://snap-ci.com), our continuous integration service. SNAP-CI uses a build pipeline concept that is very interesting and one of its pros is the faster feedback, giving the possibility of parallelism or not, and definition of steps for the total build. For more information on Build Pipeline I recommend reading [Martin Fowler's Continuous Integration article](http://www.martinfowler.com/articles/continuousIntegration.html).

We go to the SNAP-CI website, we log in and register a project. If you do not have a registration you will have to create one, but it is very fast.

![Adding repositories in Snap-CI](images/image30.png)

When registering the project it will appear in the upper part, on the right, a field with the name "CCTray" that, when we click, it directs to the XML file with the information of the build.

![Viewing pipelines created in Snap-CI](images/image32.png)

And these are the information that we will consult with our build-checker.

```xml
<Projects>
  <Project
name="brasil-de-fato/news-service (master) :: CONTRACT-TEST"
activity="Sleeping"
lastBuildLabel="77"
lastBuildStatus="Success"
lastBuildTime="2016-01-21T18:46:48Z"
webUrl="https://snap-ci.com/brasil-de-fato/news-service/branch/master/logs/defaultPipeline/77/CONTRACT-TEST"/>
</Projects>
```

Analyzing the information we realize that the only information we should validate for our build is the data from the lastBuildStatus field. It is returned if the build was successfully completed, successfully completed, or is happening at the time of validation.

The `lastBuildStatus` field can contain:

- `Success`: the last job on the server has been successfully completed;
- `Failure`: the last job on the server was terminated with error;
- `Pending`: the task is happening right now on the server;
- `Exception` and` Unknown`: Something unexpected occurred with the current task on the server. The reasons are the most diverse, as the server had a swing in the middle of the task or he never ran a certain task yet, so he does not have the information;


Let's now add our URL containing the CCTray information from our project and create it with the request for this information. For this we will add the package NodeJS [request](https://github.com/request/request), an HTTP client that was developed in order to facilitate the creation of HTTP or HTTPS requests. To add the new packages we will enter the following command.

```bash
$ npm install --save request
```

> The use of the request is very simple, accepting 2 parameters being the first the URL and the second the function that will manipulate the requested information. For more details, see [full documentation of the request package in Github](https://github.com/request/request#table-of-contents).

Now let's add the package in our project and create the request using the RequestJS module.

```javascript
...
var request = require('request');
var five = require('johnny-five');
var board = new five.Board();
...
```

And we'll add the CCTray address that we've copied from our continuous integration server into our code and create the HTTP request to read the information.

```javascript
// For educational purposes this code is using this format
// For computational resource issues use the URL as a string
// instead of manipulating the Array in the final project
var CI_CCTRACKER_URL = [
  'https://snap-ci.com',
  'willmendesneto',
  'generator-reactor',
  'branch',
  'master',
  'cctray.xml'
].join('/');
```

Let's explain a bit about RequestJS *callback*. It returns 3 parameters:

- `error`: an object with the error information that happened. If the request does not return any error it has the default value `null`;
- `response`: object with the request response information;
- `body`: String with the information of the body of the return of the requisition;

In our code then we will analyze the return of the requisition and create the appropriate treatments. The first treatment will be the verification of the error and in case we have an error we will treat the error.

```javascript
...
request(CI_CCTRACKER_URL, function(error, response, body) {
  if (error) {
    console.log('Something is wrong in our CI/CD =(');
    return;
  }
  ...
});
```

If the response does not return any error, we will treat the message to turn the LEDs on and off for visual feedback.

```javascript
...
if(body.indexOf('Failure') !== -1) {
  console.log('Your CI/CD is broken! Fix it!!!!');
  ledSuccess.off();
  ledError.on();
} else {
  console.log('Your CI/CD is ok!');
  ledSuccess.on();
  ledError.off();
}
...
```

With the request created, we will only create a gap between each request, using a simple `setInterval`. We will use a time of 500 milliseconds for code validation, but this value can be changed to whatever is ideal for you.

```javascript
setInterval(function(){
  request(CI_CCTRACKER_URL, function(error, response, body) {
    ..
  });
}, 500);
```

And the final content of our `src/index.js` was as follows:

```javascript
var request = require('request');
var five = require('johnny-five');
var board = new five.Board();

// For educational purposes this code is using this format
// For computational resource issues use the URL as a string
// instead of manipulating the Array in the final project
var CI_CCTRACKER_URL = [
  'https://snap-ci.com',
  'willmendesneto',
  'generator-reactor',
  'branch',
  'master',
  'cctray.xml'
].join('/');

board.on('ready', function() {

  var ledSuccess = new five.Led(12);
  var ledError = new five.Led(10);

  setInterval(function(){
    request(CI_CCTRACKER_URL, function(error, response, body) {
      if (error) {
        console.log('Something is wrong in our CI/CD =(');
        return;
      }

      if(body.indexOf('Failure') !== -1) {
        console.log('Your CI/CD is broken! Fix it!!!!');
        ledSuccess.off();
        ledError.on();
      } else {
        console.log('Your CI/CD is ok!');
        ledSuccess.on();
        ledError.off();
      }

    });
  }, 500);
});
```

It will query the data at a preconfigured interval and check the current state of the build, based on information from all pipelines. If there is no word *"Failure"* in the response to the request, something wrong has happened and our *build checker* will turn on the red light, otherwise the green light will remain on, signaling that everything is ok.


## Tuning the architecture of our application

Now that we've finished the first step and have seen our working with our components, let's think about improving our architecture.

We can see that we have some rather loose values, which do not say much if we do not access the Johnny Five framework documentation, such as numbers 12 and 10. For these and other configurations we will create a `configuration.js` file with Project information.

The initial contents of this file will be:

```javascript
// src/configuration.js
module.exports = {
  LED: {
    SUCCESS: 12,
    ERROR: 10
  },
  CI_CCTRACKER_URL: 'https://snap-ci.com/willmendesneto/generator-reactor/branch/master/cctray.xml',
  INTERVAL: 1000
};
```

Now let's change our `src/index.js` to use our file with the default settings of our application

```javascript
var five = require('johnny-five');
var CONFIG = require('./configuration');
var board = new five.Board();

board.on('ready', function() {

  var ledSuccess = new five.Led(CONFIG.LED.SUCCESS);
  var ledError = new five.Led(CONFIG.LED.ERROR);

  setInterval(function(){
    request(CONFIG.CI_CCTRACKER_URL, function(error, response, body)
    {
      ...
    });
  }, CONFIG.INTERVAL);
});
```

Our code is starting to get a little more expressive, do you agree? But is there still something we can improve on? Of course yes!

We are talking about *build checker*, but we have no abstraction for this operation. The idea is that our final code is just an initialization of our app, with all relevant information within this abstraction.

Let's then create our file with the LED information abstractions and the HTTP request by accessing the settings.

```javascript
var CONFIG = require('./configuration');
var request = require('request');
var five = require('johnny-five');

intervalId = null;
function BuildChecker() {
  this.ledSuccess = new five.Led(CONFIG.LED.SUCCESS);
  this.ledError = new five.Led(CONFIG.LED.ERROR);
};

BuildChecker.prototype.stopPolling = function() {
  clearInterval(intervalId);
};

BuildChecker.prototype.startPolling = function() {
  var self = this;

  intervalId = setInterval(function(){
    request.get(CONFIG.CI_CCTRACKER_URL, function(error, response, body) {
      if (error) {
        console.log('Somethink is wrong with your CI =(');
        return;
      }

      if(body.indexOf('Success') !== -1) {
        console.log('Your CI is ok!');
        self.ledSuccess.on();
        self.ledError.off();
      } else {
        console.log('Somethink is wrong with your CI =(. Fix it!!!!');
        self.ledSuccess.off();
        self.ledError.on();
      }

    });
  }, CONFIG.INTERVAL);
};

module.exports = BuildChecker;
```

And our `src/index.js` file will only invoke and start our code so that the LEDs start flashing.

```javascript
var BuildChecker = require('./build-checker');
var five = require('johnny-five');
var board = new five.Board();

board.on('ready', function() {
  buildChecker = new BuildChecker();
  buildChecker.startPolling();
});
```

Finishing this separation of concepts, we improve the readability, maintainability and several other variants of our application. It is worth mentioning that this is a good practice and that, in the course of the book, we will always be thinking about improvements to our final code.


## Creating unit tests for build checker


This time something simple, but without good information about it is like adding drive tests on Nodebots applications. Unit testing is not something new, but you do not find content on this topic in Arduino, robots and hardware applications open easily, so we'll cover a bit on this topic in this book.

Unit testing is just one of several ways to test your software and have a reliability in the end product. Based on the [Test Pyramid](http://martinfowler.com/bliki/TestPyramid.html), this is the way you should organize the testing of your application.

![Pyramid of tests](images/image36.png)

We'll talk only about unit tests, if you'd like to know more about all layers, read the [post "TestPyramid"](http://martinfowler.com/bliki/TestPyramid.html) by Martin Fowler.

The idea of ​​the unit test is to validate and certify that your code is doing what it intends to do, giving feedback on the errors quickly before we deploy our project to production.

One aspect that nobody explains very well is about the tests in Nodebots, which has as main objective in this case to create the electrical simulations and with mocks and stubs thus simulating the communication between components.

Let's now create a folder for our unit tests with the name `test`.

```bash
$ mkdir test
```

 The unit tests will use the test framework [MochaJS](https://mochajs.org), [SinonJS](http://sinonjs.org) for *spies*, *stubs* and *mocks* and [ShouldJS]( Https://shouldjs.github.io) for *assertions*. Let's then install these packages as a dependency of project development.

 ```bash
 $ npm install --save-dev mocha sinon should
 ```

A fundamental NodeJS package in this step is [mock-Firmata](https://github.com/rwaldron/mock-firmata), created by Rick Waldron to make testing setup in Johnny-Five applications easier. The integration is very simple, you just need to load and create your fake component of the board in the test, as we can see in our setup file of `test/spec-helper.js` tests.

```javascript
require('should');
var mockFirmata = require('mock-firmata');
var five = require('johnny-five');

var board = new five.Board({
  io: new mockFirmata.Firmata(),
  debug: false,
  repl: false
});
```

Let's then add a simple test to check the integration of our tests. First we will create a file with some MochaJS settings inside the `test` folder. This will be the initial content of our `mocha.opts`.

```bash
--reporter spec
--recursive
--require test/spec-helper.js
--slow 1000
--timeout 5000
```

A quick explanation of the configuration information used:

`--reporter spec`: A type of  reporter* used to display messages of test information;
`--recursive`: flag to identify that the tests should run recursively inside the folder;
`--require` test/spec-helper.js: *setup* file to be loaded before running the unit tests;
`--low 1000`: Maximum time in milliseconds of tolerance between tests. If this time exceeds this time will be shown the total time of that test with a differentiated color so that we can make the necessary changes;
`--timeout 5000`: Maximum time in milliseconds of tolerance for the completion of each assertion. If this time exceeds this time our tests will return with an error message;

We will create a file with the name `test/index.js` with a fairly simple assertion.

```javascript
describe('Test validation', function() {
  it('1 + 1 = 2', function(){
    (1 + 1).should.be.equal(2);
  });
});
```

Our tests use the `describe` and` it` scope methods. `Describe` is used to group scenarios, in our case` Test validation`. `It` is the identification of one of the test points. Note also that we have the `should.be.equal` method that will compare if the first value is equal to the second.

![Configuring the test suite in the repository](images/image16.png)

Let's go to our prompt/terminal and enter the following command.

```bash
$ ./node_modules/bin/mocha
```

We will then see this information at our prompt/terminal and our initial setup was a success!

Now, let's create the scenarios for our tests. Let us then define the scenarios that we must cover in our tests:

- Initial information when creating the `BuildChecker` instance;
- When we start *polling* and the server sends information from a successfully completed build;
- When we start *polling* and the server sends information from a finished build with failures;
- When we stop our *polling* and we will not do more requests of data for our server;

One way to validate when the build checker should flash the LED is to create a stub for the request using the node-request to validate the response by expected types (`success` and` error`) and use some spies for the LEDs.

For this simulation we will create some *fixtures* with the server responses model for success and error. Let's then create our folder with the data inside our folder containing our tests.

```bash
$ mkdir test/fixtures
$ touch test/fixtures/success.xml test/fixtures/error.xml
```

And we'll add the information for each file.

```xml
<!-- test/fixtures/error.xml -->
<Projects>
  <Project
    name="Error-project"
    activity="Sleeping"
    lastBuildLabel="22"
    lastBuildStatus="Failure"
    lastBuildTime="2016-01-04T02:20:25Z"
    webUrl="https://google.com"/>
</Projects>
```

```xml
<!-- test/fixtures/success.xml -->
<Projects>
  <Project
    name="Success-project"
    activity="Sleeping"
    lastBuildLabel="36"
    lastBuildStatus="Success"
    lastBuildTime="2016-03-22T21:09:02Z"
    webUrl="https://google.com"/>
</Projects>
```

Let's then create the scenario to validate our code. Some things we should keep in mind about our unit testing framework is that its `beforeEach` method, which happens before every` it` method. We will use as documentation of each step that must occur to reproduce the specific scenario.

Let's then explain more about the contents of this file and why of each test. We create the tests of the instance of our `BuildChecker` and its initial attributes.

```javascript
var BuildChecker = require('../src/build-checker');
var five = require('johnny-five');
var request = require('request');
var sinon = require('sinon');

describe('BuildChecker', function() {

  beforeEach(function(){
    buildChecker = new BuildChecker();
  });

  it('should have the led success port configured', function(){
    (buildChecker.ledSuccess instanceof five.Led).should.be.equal(true);
  });

  it('should have the led error port configured', function(){
    (buildChecker.ledError instanceof five.Led).should.be.equal(true);
  });

});
```

Now we will validate when we stop our polling. Let's now use the spy method of the sinon to check if the code used the clearInterval method to end with the requests. For this we will check if the `global.clearInterval` was used once, by accessing the boolean` calledOnce`, which is an internal counter added by the `sinon.spy` method for the tests.

```javascript
...
describe('#stopPolling', function(){
  beforeEach(function(){
    sinon.spy(global, 'clearInterval');
    buildChecker.stopPolling();
  });

  it('should remove interval', function(){
    global.clearInterval.calledOnce.should.be.true;
  });
});
...
```

And now the server scenarios responding successfully and failed. For this we will assign our data from the * fixtures * folder to variables.

```javascript
...
var fs = require('fs');
var successResponseCI = fs.readFileSync(__dirname + '/fixtures/success.xml', 'utf8');
var errorResponseCI = fs.readFileSync(__dirname + '/fixtures/error.xml', 'utf8');
...
```

Note that in each of the success and failure cases we are using the `sinon.useFakeTimers` method, which is a way to simulate events linked to timer objects in Javascript.

When we call the `clock.tick` method with the information contained in the configuration file, we simulate time and time changes at the time of testing, which helps us force the call to the polling event, which uses` setInterval`.

```javascript
...
clock = sinon.useFakeTimers();
clock.tick(CONFIG.INTERVAL);
...
```

We now use the syntax stub method for the `node-request` package. With this we can change the return when the `request.get` method is called. In this case we can simulate the response of each request, based on the information of our fixtures.

```javascript
...
sinon.stub(request, 'get').yields(null, null, successResponseCI);
...
```

An important point in our tests is to remember to restore all the `stub` and` fakeTimers` information. We will use the `afterEach` method that is always called after each` it` method, and we will add our objects by calling the restore method added by `sinon`.

```javascript
...
afterEach(function(){
  request.get.restore();
  clock.restore();
});
...
```

Based on our test scenarios, this is the test content of our build checker file:

```javascript
// test/build-checker.js

var BuildChecker = require('../src/build-checker');
var CONFIG = require('../src/configuration');
var five = require('johnny-five');
var request = require('request');
var sinon = require('sinon');
var fs = require('fs');
var successResponseCI = fs.readFileSync(__dirname + '/fixtures/success.xml', 'utf8');
var errorResponseCI = fs.readFileSync(__dirname + '/fixtures/error.xml', 'utf8');
var clock = null;

describe('BuildChecker', function() {

  beforeEach(function(){
    buildChecker = new BuildChecker();
  });

  it('should have the led success port configured', function(){
    (buildChecker.ledSuccess instanceof five.Led).should.be.equal(true);
  });

  it('should have the led error port configured', function(){
    (buildChecker.ledError instanceof five.Led).should.be.equal(true);
  });

  describe('#stopPolling', function(){
    beforeEach(function(){
      sinon.spy(global, 'clearInterval');
      buildChecker.stopPolling();
    });

    it('should remove interval', function(){
      global.clearInterval.calledOnce.should.be.true;
    });
  });

  describe('#startPolling', function(){
    beforeEach(function(){
      sinon.spy(global, 'setInterval');
      buildChecker.startPolling();
    });

    afterEach(function(){
      global.setInterval.restore();
    });

    it('should creates polling', function(){
      global.setInterval.calledOnce.should.be.true;
    });

    describe('When the CI server send success response', function(){
      beforeEach(function() {
        clock = sinon.useFakeTimers();
        sinon.stub(request, 'get').yields(null, null, successResponseCI);
        sinon.spy(buildChecker.ledSuccess, 'on');
        sinon.spy(buildChecker.ledError, 'off');
        buildChecker.startPolling();
        clock.tick(CONFIG.INTERVAL);
      });

      afterEach(function(){
        request.get.restore();
        clock.restore();
      });

      it('should turn on the success led', function(){
        buildChecker.ledSuccess.on.calledOnce.should.be.true;
      });

      it('should turn off the error led', function(){
        buildChecker.ledError.off.calledOnce.should.be.true;
      });

    });

    describe('When the CI server send error response', function(){
      beforeEach(function() {
        clock = sinon.useFakeTimers();
        sinon.stub(request, 'get').yields(null, null, errorResponseCI);
        sinon.spy(buildChecker.ledError, 'on');
        sinon.spy(buildChecker.ledSuccess, 'off');
        buildChecker.startPolling();
        clock.tick(CONFIG.INTERVAL);
      });

      afterEach(function(){
        request.get.restore();
        clock.restore();
      });

      it('should turn off the success led', function(){
        buildChecker.ledSuccess.off.calledOnce.should.be.true;
      });

      it('should turn on the error led', function(){
        buildChecker.ledError.on.calledOnce.should.be.true;
      });

    });

  });

});
```

This is just one of several unit testing formats for your application. With this we finish our first project with unit tests based on our possible scenarios, but if you want to download or fork the final code, access the [build checker project repository in Github](https://github.com/willmendesneto/build-checker). Let's go to our next project with Nodebot and Johnny-five?
