# ionicapidemo2

This project was generated with the [Angular Full-Stack Generator](https://github.com/DaftMonk/generator-angular-fullstack) version 4.1.0.

## CORS Modifications
  /server/config/express.js
  
  
  ```typescript
     /**
     * Express configuration
     */

    'use strict';

    import express from 'express';
    import favicon from 'serve-favicon';
    import morgan from 'morgan';
    import shrinkRay from 'shrink-ray';
    import bodyParser from 'body-parser';
    import methodOverride from 'method-override';
    import cookieParser from 'cookie-parser';
    import errorHandler from 'errorhandler';
    import path from 'path';
    import lusca from 'lusca';
    import config from './environment';
    import passport from 'passport';
    import session from 'express-session';
    import connectMongo from 'connect-mongo';
    import mongoose from 'mongoose';
    var MongoStore = connectMongo(session);

    export default function(app) {
      var env = app.get('env');

      if(env === 'development' || env === 'test') {
        app.use(express.static(path.join(config.root, '.tmp')));
      }

      if(env === 'production') {
        app.use(favicon(path.join(config.root, 'client', 'favicon.ico')));
      }
      // Added CORS to allow all request types, DONT DO THIS IN PRODUCTION!
      var allowCrossDomain = function(req, res, next) {
        res.header('Access-Control-Allow-Origin', '*');
        res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS');
        res.header('Access-Control-Allow-Headers', 'Content-Type,Authorization,Accept,X-XSRF-TOKEN');
        next();
      }

      app.set('appPath', path.join(config.root, 'client'));
      app.use(express.static(app.get('appPath')));
      app.use(morgan('dev'));

      app.set('views', `${config.root}/server/views`);
      app.engine('html', require('ejs').renderFile);
      app.set('view engine', 'html');
      app.use(shrinkRay());
      app.use(bodyParser.urlencoded({ extended: false }));
      app.use(bodyParser.json());
      app.use(methodOverride());
      app.use(cookieParser());
      app.use(passport.initialize());
      app.use(allowCrossDomain);        //Load the CORS module into the ExpressJS app.


      // Persist sessions with MongoStore / sequelizeStore
      // We need to enable sessions for passport-twitter because it's an
      // oauth 1.0 strategy, and Lusca depends on sessions
      app.use(session({
        secret: config.secrets.session,
        saveUninitialized: true,
        resave: false,
        store: new MongoStore({
          mongooseConnection: mongoose.connection,
          db: 'ionicapidemo2'
      })
    }));

  /**
   * Lusca - express server security
   * https://github.com/krakenjs/lusca
   */
  if(env !== 'test' && !process.env.SAUCE_USERNAME) {
    app.use(lusca({
      xframe: 'SAMEORIGIN',
      hsts: {
        maxAge: 31536000, //1 year, in seconds
        includeSubDomains: true,
        preload: true
      },
      xssProtection: false
    }));
  }

  if(env === 'development') {
    const webpackDevMiddleware = require('webpack-dev-middleware');
    const stripAnsi = require('strip-ansi');
    const webpack = require('webpack');
    const makeWebpackConfig = require('../../webpack.make');
    const webpackConfig = makeWebpackConfig({ DEV: true });
    const compiler = webpack(webpackConfig);
    const browserSync = require('browser-sync').create();

    /**
     * Run Browsersync and use middleware for Hot Module Replacement
     */
    browserSync.init({
      open: false,
      logFileChanges: false,
      proxy: `localhost:${config.port}`,
      ws: true,
      middleware: [
        webpackDevMiddleware(compiler, {
          noInfo: false,
          stats: {
            colors: true,
            timings: true,
            chunks: false
          }
        })
      ],
      port: config.browserSyncPort,
      plugins: ['bs-fullscreen-message']
    });

    /**
     * Reload all devices when bundle is complete
     * or send a fullscreen error message to the browser instead
     */
    compiler.plugin('done', function(stats) {
      console.log('webpack done hook');
      if(stats.hasErrors() || stats.hasWarnings()) {
        return browserSync.sockets.emit('fullscreen:message', {
          title: 'Webpack Error:',
          body: stripAnsi(stats.toString()),
          timeout: 100000
        });
      }
      browserSync.reload();
    });
  }

  if(env === 'development' || env === 'test') {
    app.use(errorHandler()); // Error handler - has to be last
  }
}

  ```


## Getting Started

### Prerequisites

- [Git](https://git-scm.com/)
- [Node.js and npm](nodejs.org) Node >= 4.x.x, npm >= 2.x.x
- [Gulp](http://gulpjs.com/) (`npm install --global gulp`)
- [MongoDB](https://www.mongodb.org/) - Keep a running daemon with `mongod`

### Developing

1. Run `npm install` to install server dependencies.

2. Run `mongod` in a separate shell to keep an instance of the MongoDB Daemon running

3. Run `gulp serve` to start the development server. It should automatically open the client in your browser when ready.

## Build & development

Run `gulp build` for building and `gulp serve` for preview.

## Testing

Running `npm test` will run the unit tests with karma.
