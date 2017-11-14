[![Build Status](https://travis-ci.org/csgis/bricjs-loader.svg?branch=master)](https://travis-ci.org/csgis/bricjs-loader) [![codecov](https://codecov.io/gh/csgis/bricjs-loader/branch/master/graph/badge.svg)](https://codecov.io/gh/csgis/bricjs-loader) [![npm](https://img.shields.io/npm/v/@csgis/bricjs-loader.svg)](https://www.npmjs.com/package/@csgis/bricjs-loader) [![downloads](https://img.shields.io/npm/dt/@csgis/bricjs-loader.svg)](https://www.npmjs.com/package/@csgis/bricjs-loader)

# BricJS loader

BricJS is a Webpack loader to integrate reusable modules in an application.

Some restrictions must be taken into account when creating both reusable modules and applications.

## Modules

Modules must export a `bricjs` function with the following parameters:

* `props`: An object with data for customizing the module.
* `...deps`: One or more dependencies, as separated arguments.

For example:

```js
export function bricjs(props, timeService, renderer) {
  if (timeService.isAfternoon()) {
    renderer.render('Good afternoon, ' + props.name);
  } else {
    renderer.render('Hello, ' + props.name);
  }
}
```

## Apps

The BricJS loader reads a JSON file and produces a function that executes all modules, based on a given configuration(s). Let's break it down.

### app.json

First, you need to define the modules and dependencies in your application with the JSON file. It must contain an object with the following properties:

* `modules` (*Object*): keys are modules names (to be used later in the configuration); values are paths to be used in [import](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) statements. 
* `deps` (*Object*): keys are modules names (same module names as for `modules`); values are arrays containing the names of the modules to be used as dependencies when calling the `bricjs` function. Arrays must **not** include the `props` parameter; all dependencies must be in order.

For example:

```json
{
  "modules": {
    "time": "@csgis/time",
    "render": "@csgis/render",
    "greeting": "src/greeting"
  },
  "deps": {
    "greeting": ["time", "render"]
  }
}
```

**IMPORTANT**: If a module has dependencies but they are not defined in the JSON file, the names of the variables will be used as module names.

### main.js

Then you need to import your JSON file in some JavaScript module and call it with your configuration:

```js
import app from '@csgis/bricjs-loader!./app.json';
app({
  "greeting": {
    "name": "Víctor"
  }
});
```

### Configuration

The configuration passed to the app generated by the loader can be an object or an array of objects (for multiple *environments*).

For that object, keys are module names (as defined in the `app.json` file) and values are the `props` values to be passed to the modules when called.

**IMPORTANT**: Each module can be disabled (its `bricjs` function will never be called) via configuration by adding the `enabled` property with a `false` value. For example:

```json
{
  "greeting": {
    "name": "Víctor",
    "enabled": false
  }
}
```
