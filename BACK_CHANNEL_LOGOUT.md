# Back-Channel Logout (Experimental)

> Note: This is an experimental branch to test Back-Channel Logout and should not be used in production.

## Install

```bash
npm i "auth0/express-openid-connect#back-channel-logout"
```

## Basic Setup

Load the following environment variables:

```bash
ISSUER_BASE_URL=https://YOUR_DOMAIN
CLIENT_ID=YOUR_CLIENT_ID
BASE_URL=https://YOUR_APPLICATION_ROOT_URL
SECRET={A_LONG_RANDOM_VALUE}
```

Configure the SDK with `backChannelLogout` enabled. You will also need a session store (like Redis) - you can use any `express-session` compatible store.

```js
// index.js
const { auth } = require('express-openid-connect');
const { createClient } = require('redis');
const RedisStore = require('connect-redis')(auth);

// redis@v4
let redisClient = createClient({ legacyMode: true });
redisClient.connect().catch(console.error);

app.use(
  auth({
    idpLogout: true,
    backChannelLogout: true,
    backChannelLogoutStore: new RedisStore({ client: redisClient }),
  })
);
```

If you're already using a session store for stateful sessions you can just reuse that.

```js
app.use(
  auth({
    idpLogout: true,
    session: {
      store: new RedisStore({ client: redisClient }),
    },
    backChannelLogout: true,
  })
);
```

### This will:

- Create the handler `/back-channel-logout` that you can register with your ISP.
- On receipt of a valid Logout Token, the SDK will store it by `sid` (Session ID) in the `backChannelLogoutStore` (this is customisable using the [storeLogoutToken](https://github.com/auth0/express-openid-connect/blob/back-channel-logout/index.d.ts#L482) config hook)
- On all authenticated requests, the SDK will check the store for a Logout Token that corresponds with the session's ID token's `sid`. If it finds a corresponding Logout Token it will invalidate the session and clear the session cookie. (This is customisable using the [getLogoutToken](https://github.com/auth0/express-openid-connect/blob/back-channel-logout/index.d.ts#L498) config hook)

## Resources

- The config options are [documented here](https://github.com/auth0/express-openid-connect/blob/back-channel-logout/index.d.ts#L450)
- There's a [running example of the basic setup here](https://github.com/auth0/express-openid-connect/blob/back-channel-logout/examples/back-channel-logout.js)
  - run it using `npm run start:example -- back-channel-logout`
  - login to the mock Identity Provider using any credentials
  - issue a Back-Channel Logout by visiting `/logout-token` and clicking the button
- There's a [running example of a custom setup here](https://github.com/auth0/express-openid-connect/blob/back-channel-logout/examples/back-channel-logout-custom.js)
  - This custom implementation deals with IDPs that can send logout tokens with either an `sid` or a `sub` or both (The default implementation assumes you always get a `sid`)
- The specification https://openid.net/specs/openid-connect-backchannel-1_0.html
