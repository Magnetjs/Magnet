[![Build Status](https://travis-ci.org/Magnetjs/magnet-core.svg?branch=master)](https://travis-ci.org/Magnetjs/magnet-core)

### Philosophy
- Standard is god.
- It only do following:
  - Define folder structure
  - Define where to load config
  - Define how to setup and teardown modules
  - Define how to pass around module
- Everything else is up to you.

### Status
Under development, API might change for all Magnet modules.

### Module boilerplate
~~~javascript
import Base from 'magnet-core/dist/base';

export default class Module extends Base {
  init () {
    /*
      E.g. You can put @google/maps | apollo-server-express | apollo-server,
      it will format to snake case _google_maps | apollo_server_express | apollo_server.
      So we can use it as _google_maps.geocode({ address: '1600 Amphitheatre Parkway, Mountain View, CA' }).
      It's a Magnet practice to use npm module name for scoping, to reduce clash on namespace
    */
    this.moduleName = '<module npm name>'

    // Optional, path of default config folder
    this.defaultConfig = __dirname
  }

  /**
   * Code to setup the module, either:
   * - Add module to this.app object
   * - Add middleware to this.app.application
   */
  async setup() {
    this.app.module = {};
  }

  /**
   * How to handle shutdown
   * Not implement yet
   */
  async teardown() {
    this.app.module = {};
  }
}
~~~

### Folders structure
- client
- docs
- universal
- local_modules
- server
  - config
  - controllers
  - emails
  - graphql
  - queues
  - schedulers
- static
- tests

### App structure
~~~javascript
App = {
  config: // magnet-config
};
~~~

### Modules
Allow Magnet module is searchable under magnetjs keywords
[NPM search](https://www.npmjs.com/search?q=magnetjs)

### Usage
es6
~~~javascript
import magnet from 'magnet-core';
import Config from 'magnet-config';
import Logger from 'magnet-bunyan';
import Router from 'magnet-router';
import FileLoader from '../local_modules/file_loader';

magnet([
  Config,
  Logger,
  Router,
  {
    module: FileLoader,
    options: ''
  }
]);
~~~

es5
~~~javascript
var magnet = require('magnet-core').default,
var Config = ,
var Logger = ,
var Router = ,
var FileLoader = require('../local_modules/file_loader').default,

magnet([
  require('magnet-config').default,
  require('magnet-bunyan').default,
  require('magnet-router').default,
  {
    module: FileLoader,
    options: ''
  }
]);
~~~

Magnet style
~~~javascript
import magnet, { from, fromLocal } from 'magnet-core';

magnet([
  from('magnet-config'),
  from('magnet-bunyan'),
  from('magnet-router'),
  fromLocal('file_loader'),
]);
~~~

### Example
Scheduler Server
~~~javascript
import magnet from 'magnet-core';
import Config from 'magnet-config';
import Logger from 'magnet-bunyan';
import Kue from 'magnet-kue';

magnet([
  Config,
  Logger,
  Kue
]);
~~~

API Server
~~~javascript
import magnet from 'magnet-core';
import Config from 'magnet-config';
import Logger from 'magnet-bunyan';
import Spdy from 'magnet-spdy';
import Common from 'magnet-server-common';
import Helmet from 'magnet-helmet';
import Router from 'magnet-router';
import Controller from 'magnet-controller';
import Mongoose from 'magnet-mongoose';
import Session from 'magnet-redis-session';
import Respond from 'magnet-respond';
import FileLoader from '../local_modules/file_loader';

magnet([
  Config,
  Logger,
  Spdy,
  [
    Respond,
    Session,
    Common,
    Helmet,
    Router,
    Mongoose
  ],
  Controller,
  {
    module: FileLoader,
    options: ''
  }
]);
~~~

Scheduler Server
~~~javascript
import magnet, { from, fromLocal } from 'magnet-core';

magnet([
  fromM('config'),
  fromM('bunyan'),
  fromM('file_loader'),
  fromM('sequelize'),
  fromM('sequelize', {
    namespace: 'sequelizeAnotherDb',
    database: 'anotherDb'
  })
]);

// app.sequelize.query(...)
// app.sequelizeAnotherDb.query(...)
~~~

### Multiple instance

~~~javascript
import magnet from 'magnet-core';
import Config from 'magnet-config';
import Logger from 'magnet-bunyan';
import Kue from 'magnet-kue';

magnet([
  Config,
  Logger,
  Kue
]);
~~~

### Naming
All magnet module is store under app variables.
To avoid conflict some rules is introduced.
- magmet, config, log is reserved
- npm organization scope replaced with underscore

~~~javascript
// Reserved
const app = {
  magnet: null,
  config: null,
  log: null,
}

this.app.koa = new Koa()
this.app._googleMaps = require('@google/maps').createClient({
  key: 'your API key here'
})
~~~

### CLI
Magnet come with cli command to copy all config files from following to `server/config`:
- `local_modules/**/config/*.js`
- `node_modules/**/config/*.js`

Just run from `./node_modules/.bin/magnet`

Or install globally `npm install -g magnet-core` `yarn global add magnet-core`
And run `magnet`

### Roadmap
- Find solution to copy config/* without babel
- Update `config/*.js` when extra field introduced
- Make this.app = {} immutable? Only can set via Module.set('redis', redis), or map, or both

### Todo
- Why copyConfig doesn't get magnet-koa/config/koa
- Auto build modules structure via npm dependencies
