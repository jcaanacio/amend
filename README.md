# amend
Amend is a dependency-injection framework for use in node and in the browser. Amend aims to have no impact on the code you write and to minimize configuration as much as possible.

## installation
```bash
npm i amend -S
```
or
```
npm install amend --save
```
# Current state
Amend is in alpha mode. The dependency injector works but the project is not fully tested for the full development cycle, i.e work in development mode, packge and deploy to production. This is expected to be done in december 2014 since amend will be developed and used together with a production ready project by then.

# environments
  * Works in node
  * Works minified by using amend-annotate, see section tools for more information.

# introduction
Instead of forcing you to use code specific to this library when creating your components, amend lets you write components using the regular ```require``` and ```module.exports``` function and variable. Depenencies between the components are wired up through a configuration object. This means that unlike many other di frameworks, no variables specific for amend are introduced in your code. 

The configuration is a normal javascript object that maps names of components to their path for require. A module can be one of these three types:
### factory
Factories are functions that can have dependencies injected in them. The factory function is executed by amend and whatever the factory returns is what you get when you access the module through amend. The dependency arguments should correspond to the names of the modules that are to be injected.
```javascript

module.exports = function(dependency1, dependency2, ...) {
  return //returned function/object
}
```
### class
Classes are similair to factories in that they can have dependencies injected in them. But the function is treated like a constuctor and is therefore invoked with ```new```. This means that you can assign values to ```this``` and access them in your created module.
```javascript
module.exports = function(dependency1, dependency2, ...) {
  this.foo = something;
}
```
### value
Values are simpler than factories and constructors, they cannot have dependencies themselves. Whatever you assign to module.exports will be what you get when you access the module through amend.
```javascript
module.exports = //returned function/object
```

# API
## amend.fromConfig(config, basePath, options)
Returns a di container with modules loaded acording to the specified configuration.

```config``` is a normal javascript object that contains the key ```modules``` which keys are the names of the modules. A module can have two configuration options.
```require``` is the path by which the module will be required. Require paths can be realtive or to an installed dependency, relative paths should be relative to the root directory of the poject.
```type```  is either "factory", "class" or "value".
```javascript
  var conf = {
    modules: {
      foo: {
         require: './lib/foo',
         type: 'factory'
      }
    }
  }
```
It is also possible to use "short" notation where only the path is configured:
```javascript
  var conf = {
    modules: {
      foo: './lib/foo'
    }
  }
```
When using "short" notation the module is assumed to be a factory if it returns a function, otherwise it is treated as a value. This means that it is only neccesary to use "long" notation when registering a class or when the module consists of a function that should be treated as a value.


```basePath``` should be the root dir of the project using amend.

```options``` is passed to the created container.

## new Container(options)
Manually create a container. This is not the recommended way to do it as a container is created and populated with ```amend.fromConfig```. The container constuctor is required by ```reuire('amend').Container```.

## container.get(name) 
Returns the component with the specified name. If the component is a factory or a class it will be initialized on the first call to get. On subsquent calls the initialized component will be returned.

## container.initializeAll()
All components are lazy loaded, only initialized when they are required. This method will initialize all modules however. If there are any missmatching dependencies in the container, a call to this method will detect them by throwing an error.

## container.factory(name, factoryFn)
Manually register a factory.
## container.class(name, constructorFn)
Manually register a class.
## container.value(name, value)
Manually register a value.

## example
A little dice game:
```json
  // file: ./config.json
  {
    "modules": {
      "rollDice": "./roll-dice.js",
      "playGame": "./play-game.js",
      "_": {
        "require": "lodash",
        "type:": "value"
      }
    }
  }
```
```javascript
  // file: ./roll-dice.js
  module.exports = function(_) {
    return function() {
      return _.random(1, 6);
    };
  };
```
```javascript
  // file: ./playGame.js
  module.exports = function(rollDice) {
    return function() {
      return "The roll was " + rollDice()
    };
  };
```
```javascript
  //file ./main.js
  var amend = require('amend');
  var config = require('./config.json');

  var di = amend.fromConfig(config);

  var playGame = di.get('playGame');
  console.log(playGame()); // prints "The roll was 4"
```
# tools
## annotate
Annotate takes an amend configuration and produces an output which describes the dependencies between the different components. This output can then be used when creating a container in a minified enviroment where the names of dependencies have been obfuscated. Annotate is included as an executable and can thus be accessed through adding an npm script, through the ```node_modules/.bin``` folder or if amend is installed globally. To execute the binary, run:
```bash
ammend-annotate config.json
```
This will print the output to standard out.

It is also possible to use the api:
```javascript
var annotate = require('amend/tools').annotate;
var config = require('./my-config.json');
var di = amend.fromConfig(config);
var out = getMyWriteStream()

annotate(di, out);
```
Use the annotated output when creating your container. If you have written the output to ```./dependencies.json``` then create a container like this:
```javascript
var amend = require('amend');
var config = require('./config.json');
var dependencies = require('./dependencies.json');
var opts = {
  modules: dependencies
};
var di = amend.fromConfig(config, opts);
```
