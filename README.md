everyauth
==========

Authentication and authorization (password, facebook, & more) for your node.js Connect and Express apps.

So far, `everyauth` enables you to login via:

- `password`
- `facebook`
- `twitter`
- `github`
- `instagram`
- `foursquare`
- `linkedin`

`everyauth` is:

- **Modular** - We have you covered with Facebook and Twitter 
  OAuth logins, basic login/password support, and modules 
  coming soon for beta invitation support and more.
- **Easily Configurable** - everyauth was built with powerful
  configuration needs in mind. Configure an authorization strategy 
  in a straightforward, easy-to-read & easy-to-write approach, 
  with as much granularity as you want over the steps and 
  logic of your authorization strategy.
- **Idiomatic** - The syntax for configuring and extending your authorization strategies are
  idiomatic and chainable.


## Installation
    $ npm install everyauth


## Quick Start
Using everyauth comes down to just 2 simple steps if using Connect
or 3 simple steps if using Express:

1. **Choose and Configure Auth Strategies** - Find the authentication strategy
   you desire in one of the sections below. Follow the configuration
   instructions.
2. **Add the Middleware to Connect**
        
    ```javascript
    var everyauth = require('everyauth');
    // Step 1 code goes here

    // Step 2 code
    var connect = require('connect');
    var app = connect(
        connect.bodyParser()
      , connect.cookieParser()
      , connect.session({secret: 'mr ripley'})
      , everyauth.middleware()
      , connect.router(routes)
    );
    ```
3. **Add View Helpers to Express**
    
    ```javascript        
    // Step 1 code
    // ...
    // Step 2 code
    // ...

    // Step 3 code
    everyauth.helpExpress(app);

    app.listen(3000);
    ```
    
    For more about what view helpers `everyauth` adds to your app, see the section
    titled "Express Helpers" near the bottom of this README.

## Example Application

There is an example application at [./example](https://github.com/bnoguchi/tree/master/example)

To run it:

    $ cd example
    $ node server.js

Some OAuth Providers do not allow callbacks to localhost, so you will need to create a `localhost`
alias called `local.host`. Make sure you set up your /etc/hosts so that 127.0.0.1 is also 
associated with 'local.host'. So inside your /etc/hosts file, one of the lines will look like:

    127.0.0.1	localhost local.host

Then point your browser to [http://local.host:3000](http://local.host:3000)

## Logging Out

If you integrate `everyauth` with `connect`, then `everyauth` automatically
sets up a `logoutPath` at `GET` `/logout` for your app. It also
sets a default handler for your logout route that clears your session
of auth information and redirects them to '/'.

To over-write the logout path:

```javascript
everyauth.everymodule.logoutPath('/bye');
```

To over-write the logout redirect path:

```javascript
everyauth.everymodule.logoutRedirectPath('/navigate/to/after/logout');
```

To over-write the logout handler:

```javascript
everyauth.everymodule.handleLogout( function (req, res) {
  // Put you extra logic here
  
  req.logout(); // The logout method is added for you by everyauth, too
  
  // And/or put your extra logic here
  
  res.writeHead(303, { 'Location': this.logoutRedirectPath() });
  res.end();
});
```

## Setting up Facebook Connect

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.facebook
  .myHostname('http://localhost:3000')
  .appId('YOUR APP ID HERE')
  .appSecret('YOUR APP SECRET HERE')
  .handleAuthCallbackError( function (req, res) {
    // If a user denies your app, Facebook will redirect the user to
    // /auth/facebook/callback?error_reason=user_denied&error=access_denied&error_description=The+user+denied+your+request.
    // This configurable route handler defines how you want to respond to
    // that.
    // If you do not configure this, everyauth renders a default fallback
    // view notifying the user that their authentication failed and why.
  })
  .findOrCreateUser( function (session, accessToken, fbUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.facebook
  .entryPath('/auth/facebook')
  .callbackPath('/auth/facebook/callback')
  .scope('email')                // Defaults to undefined
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.facebook.scope(); // undefined
everyauth.facebook.entryPath(); // '/auth/facebook'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.facebook.configurable();
```

#### Dynamic Facebook Connect Scope

Facebook provides many different 
[permissions](http://developers.facebook.com/docs/authentication/permissions/)
for which your app can ask your user. This is bundled up in the `scope` query
paremter sent with the oauth request to Facebook. While your app may require 
several different permissions from Facebook, Facebook recommends that you only
ask for these permissions incrementally, as you need them. For example, you might
want to only ask for the "email" scope upon registration. At the same time, for
another user, you may want to ask for "user_status" permissions because they
have progressed further along in your application.

`everyauth` enables you to specify the "scope" dynamically with a second
variation of the configurable `scope`. In addition to the first variation
that looks like:

```javascript
everyauth.facebook
  .scope('email,user_status');
```

you can have greater dynamic control over "scope" via the second variation of `scope`:

```javascript
everyauth.facebook
  .scope( function (req, res) {
    var session = req.session;
    switch (session.userPhase) {
      case 'registration':
        return 'email';
      case 'share-media':
        return 'email,user_status';
    }
  });

```

## Setting up Twitter OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.twitter
  .myHostname('http://localhost:3000')
  .consumerKey('YOUR CONSUMER ID HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (session, accessToken, accessTokenSecret, twitterUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```
      

## Setting up Password Authentication

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.password
  .getLoginPath('/login') // Page with the login form
  .postLoginPath('/login') // What you POST to
  .loginView('a string of html; OR the name of the jade/etc-view-engine view')
  .authenticate( function (login, password) {
    // Returns a user if we can authenticate with the login + password.
    // If we cannot, returns null/undefined
  })

  .getRegisterPath('/register')
  .postRegisterPath('/register')
  .registerView('a string of html; OR the name of the jade/etc-view-engine view')
  .registerUser( function (login, password) {
    // Returns a user (or a Promise that promises a user) after adding it to
    // some user store. You can also do things here like registration validation
    // and re-directing back to the registration page upon invalid registration
  })

  
  .redirectPath('/'); // Where to redirect to after a login

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

## Setting up GitHub OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.github
  .myHostname('http://localhost:3000')
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .findOrCreateUser( function (session, accessToken, githubUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:
  
```javascript  
everyauth.github
  .entryPath('/auth/github')
  .callbackPath('/auth/github/callback')
  .scope('repo'); // Defaults to undefined
                  // Can be set to a combination of: 'user', 'public_repo', 'repo', 'gist'
                  // For more details, see http://develop.github.com/p/oauth.html
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.github.scope(); // undefined
everyauth.github.entryPath(); // '/auth/github'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.github.configurable();
```

## Setting up Instagram OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.instagram
  .myHostname('http://localhost:3000')
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .findOrCreateUser( function (session, accessToken, instagramUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.instagram
  .entryPath('/auth/instagram')
  .callbackPath('/auth/instagram/callback')
  .scope('basic') // Defaults to 'basic'
                  // Can be set to a combination of: 'basic', 'comments', 'relationships', 'likes'
                  // For more details, see http://instagram.com/developer/auth/#scope
  .display(undefined); // Defaults to undefined; Set to 'touch' to see a mobile optimized version
                       // of the instagram auth page
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.instagram.callbackPath(); // '/auth/instagram/callback'
everyauth.instagram.entryPath(); // '/auth/instagram'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.instagram.configurable();
```

## Setting up Foursquare OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.foursquare
  .myHostname('http://localhost:3000')
  .appId('YOUR CLIENT ID HERE')
  .appSecret('YOUR CLIENT SECRET HERE')
  .findOrCreateUser( function (session, accessToken, foursquareUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.foursquare
  .entryPath('/auth/foursquare')
  .callbackPath('/auth/foursquare/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.foursquare.callbackPath(); // '/auth/foursquare/callback'
everyauth.foursquare.entryPath(); // '/auth/foursquare'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.foursquare.configurable();
```

## Setting up LinkedIn OAuth

```javascript
var everyauth = require('everyauth')
  , connect = require('connect');

everyauth.linkedin
  .myHostname('http://localhost:3000')
  .consumerKey('YOUR CONSUMER ID HERE')
  .consumerSecret('YOUR CONSUMER SECRET HERE')
  .findOrCreateUser( function (session, accessToken, accessTokenSecret, linkedinUserMetadata) {
    // find or create user logic goes here
  })
  .redirectPath('/');

var routes = function (app) {
  // Define your routes here
};

connect(
    connect.bodyParser()
  , connect.cookieParser()
  , connect.session({secret: 'whodunnit'})
  , everyauth.middleware()
  , connect.router(routes);
).listen(3000);
```

You can also configure more parameters (most are set to defaults) via
the same chainable API:

```javascript    
everyauth.linkedin
  .entryPath('/auth/linkedin')
  .callbackPath('/auth/linkedin/callback');
```

If you want to see what the current value of a
configured parameter is, you can do so via:

```javascript
everyauth.linkedin.callbackPath(); // '/auth/linkedin/callback'
everyauth.linkedin.entryPath(); // '/auth/linkedin'
```

To see all parameters that are configurable, the following will return an
object whose parameter name keys map to description values:

```javascript
everyauth.linkedin.configurable();
```

## Accessing the User

If you are using `express` or `connect`, then `everyauth` 
provides an easy way to access the user as:

- `req.user` from your app server
- `everyauth.user` via the `everyauth` helper accessible from your `express` views.
- `user` as a helper accessible from your `express` views

To access the user, configure `everyauth.everymodule.findUserById`.
For example, using [mongoose](http://github.com/LearnBoost/mongoose):

```javascript
everyauth.everymodule.findUserById( function (userId, callback) {
  User.findById(userId, callback);
  // callback has the signature, function (err, user) {...}
});
```

Once you have configured this method, you now have access to the user object
that was fetched anywhere in your server app code as `req.user`. For instance:

```javascript
var app = require('express').createServer()

// Configure your app

app.get('/', function (req, res) {
  console.log(req.user);  // FTW!
  res.render('home');
});
```

Moreover, you can access the user in your views as `everyauth.user`.

    //- Inside ./views/home.jade
    span.user-id= everyauth.user.name
    #user-id= user.id

## Express Helpers

If you are using express, everyauth comes with some useful dynamic helpers.
To enable them:

```javascript
var express = require('express')
  , everyauth = require('everyauth')
  , app = express.createServer();

everyauth.helpExpress(app);
```

Then, from within your views, you will have access to the following helpers methods
attached to the helper, `everyauth`:

- `everyauth.loggedIn`
- (more - we copy over req.session.auth keys/values to the everyauth helper)


## Configuring a Module

everyauth was built with powerful configuration needs in mind.

Every module comes with a set of parameters that you can configure
directly. To see a list of those parameters on a per module basis, 
with descriptions about what they do, enter the following into the 
node REPL (to access the REPL, just type `node` at the command line)

    > var ea = require('everyauth');
    > ea.facebook.configurable();

For example, you will see that one of the configuration parameters is
`moduleTimeout`, which is described to be `how long to wait per step
before timing out and invoking any timeout callbacks`

Every configuration parameter corresponds to a method of the same name
on the auth module under consideration (i.e., in this case
`ea.facebook`). To create or over-write that parameter, just
call that method with the new value as the argument:

```javascript
ea.facebook
  .moduleTimeout( 4000 ); // Wait 4 seconds before timing out any step
                          // involved in the facebook auth process
```

Configuration parameters can be scalars. But they can be anything. For
example, they can also be functions, too. The facebook module has a 
configurable step named `findOrCreateUser` that is described as 
"STEP FN [findOrCreateUser] function encapsulating the logic for the step
`fetchOAuthUser`.". What this means is that this configures the 
function (i.e., "FN") that encapsulates the logic of this step.

```javascript
ea.facebook
  .findOrCreateUser( function (session, accessToken, extra, oauthUser) {
    // find or create user logic goes here
  });
```

How do we know what arguments the function takes?
We elaborate more about step function configuration in our 
`Introspection` section below.


## Introspection

everyauth provides convenient methods and getters for finding out
about any module.

Show all configurable parameters with their descriptions:

```javascript
everyauth.facebook.configurable();
```

Show the value of a single configurable parameter:

```javascript
// Get the value of the configurable callbackPath parameter
everyauth.facebook.callbackPath(); // => '/auth/facebook/callback'
```

Show the declared routes (pretty printed):

```javascript
everyauth.facebook.routes;
```

Show the steps initiated by a given route:

```javascript
everyauth.facebook.route.get.entryPath.steps; 
everyauth.facebook.route.get.callbackPath.steps;
```

Sometimes you need to set up additional steps for a given auth
module, by defining that step in your app. For example, the
set of steps triggered when someone requests the facebook
module's `callbackPath` contains a step that you must define
in your app. To see what that step is, you can introspect
the `callbackPath` route with the facebook module.

```javascript
everyauth.facebook.route.get.callbackPath.steps.incomplete;
// => [ { name: 'findOrCreateUser',
//        error: 'is missing: its function' } ]
```

This tells you that you must define the function that defines the
logic for the `findOrCreateUser` step. To see what the function 
signature looks like for this step:

```javascript
var matchingStep =
everyauth.facebook.route.get.callbackPath.steps.filter( function (step) {
  return step.name === 'findOrCreateUser';
})[0];
// { name: 'findOrCreateUser',
//   accepts: [ 'session', 'accessToken', 'extra', 'oauthUser' ],
//   promises: [ 'user' ] }
```

This tells you that the function should take the following 4 arguments:

```javascript
function (session, accessToken, extra, oauthUser) {
  ...
}
```

And that the function should return a `user` that is a user object or
a Promise that promises a user object.

```javascript
function (session, accessToken, extra, oauthUser) {
  ...
  return { id: 'some user id', username: 'some user name' };
}

// OR

function (session, accessToken, extra, oauthUser) {
  var promise = new everyauth.Promise();
  asyncFindUser( function (err, user) {
    if (err) return promise.fail(err);
    promise.fulfill(user);
  });
  return promise;
}
```

You add this function as the block for the step `findOrCreateUser` just like
you configure any other configurable parameter in your auth module:

```javascript
everyauth.facebook
  .findOrCreateUser( function (session, accessToken, extra, oauthUser) {
    // Logic goes here
  });
```

There are also several other introspection tools at your disposal:

For example, to show the submodules of an auth module by name:

```javascript
everyauth.oauth2.submodules;
```

Other introspection tools to describe (explanations coming soon):

- *Invalid Steps*
    
    ```javascript    
    everyauth.facebook.routes.get.callbackPath.steps.invalid
    ```

## Modules and Projects that use everyauth

Currently, the following module uses everyauth. If you are using everyauth
in a project, app, or module, get in touch to get added to the list below:

- [mongoose-auth](https://github.com/bnoguchi/mongoose-auth) Authorization plugin
  for use with the node.js MongoDB orm.

### License
MIT License

---
### Author
Brian Noguchi
