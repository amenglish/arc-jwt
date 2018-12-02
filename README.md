# arc-jwt


Middleware that validates JsonWebTokens and sets `req.user`.

This module lets you authenticate HTTP requests using JWT tokens in your arc.codes
applications.  JWTs are typically used to protect API endpoints, and are
often issued using OpenID Connect.

## Usage

The JWT authentication middleware authenticates callers using a JWT.
If the token is valid, `req.user` will be set with the JSON object decoded
to be used by later middleware for authorization and access control.

For example,

```javascript
var jwt = require('arc-jwt');

arc.http(jwt({secret: 'shhhhhhared-secret'}),
  function(req, res) {
    if (!req.user.admin) return res.sendStatus(401);
    res.sendStatus(200);
  });
```

You can specify audience and/or issuer as well:

```javascript
jwt({ secret: 'shhhhhhared-secret',
  audience: 'http://myapi/protected',
  issuer: 'http://issuer' })
```

> If the JWT has an expiration (`exp`), it will be checked.

If you are using a base64 URL-encoded secret, pass a `Buffer` with `base64` encoding as the secret instead of a string:

```javascript
jwt({ secret: new Buffer('shhhhhhared-secret', 'base64') })
```

This module also support tokens signed with public/private key pairs. Instead of a secret, you can specify a Buffer with the public key

```javascript
var publicKey = fs.readFileSync('/path/to/public.pub');
jwt({ secret: publicKey });
```

By default, the decoded token is attached to `req.user` but can be configured with the `requestProperty` option.


```javascript
jwt({ secret: publicKey, requestProperty: 'auth' });
```

The token can also be attached to the `result` object with the `resultProperty` option. This option will override any `requestProperty`.

```javascript
jwt({ secret: publicKey, resultProperty: 'locals.user' });
```

Both `resultProperty` and `requestProperty` utilize [lodash.set](https://lodash.com/docs/4.17.2#set) and will accept nested property paths.

A custom function for extracting the token from a request can be specified with
the `getToken` option. This is useful if you need to pass the token through a
query parameter or a cookie. You can throw an error in this function and it will
be handled by `express-jwt`.

```javascript
arc.http(jwt({
  secret: 'hello world !',
  credentialsRequired: false,
  getToken: function fromHeaderOrQuerystring (req) {
    if (req.headers.authorization && req.headers.authorization.split(' ')[0] === 'Bearer') {
        return req.headers.authorization.split(' ')[1];
    } else if (req.query && req.query.token) {
      return req.query.token;
    }
    return null;
  }
}));
```
