# ionicapidemo2

This project was generated with the [Angular Full-Stack Generator](https://github.com/DaftMonk/generator-angular-fullstack) version 4.1.0.

## CORS Modifications
  /server/config/express.js
  
  
  ```typescript
     /**
     * Express configuration
     */

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

      app.use(session({
        // Removed Angular 1.5 CSRF Token
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
        xssProtection: false          // Disabled xssProtection
      }));
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
