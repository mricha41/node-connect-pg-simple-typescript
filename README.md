# node-connect-pg-simple-typescript
Session store based on connect-pg-simple written in typescript.

## How to Use
This session store is intended as a drop-in replacement for connect-pg-simple. First set up a database similar to [https://github.com/voxpelli/node-connect-pg-simple/blob/main/table.sql](https://github.com/voxpelli/node-connect-pg-simple/blob/main/table.sql) with the following naming differences:

* database name should be "sessions"
* "sid" is now "session_id"
* "sess" is now "session_data"
* "expire" is now "expiration"

Here is what the SQL might look like for setting up the table:

```
CREATE TABLE "sessions"
(
    session_id varchar NOT NULL COLLATE "default",
    session_data json NOT NULL,
    expiration timestamp(6) NOT NULL,
    CONSTRAINT session_pkey PRIMARY KEY (session_id)
)
WITH (OIDS=FALSE);

CREATE INDEX "IDX_session_expire" ON "sessions" ("expiration");
```

Once you have the database set up, you can set up the client connection with <code>pg.Pool</code> and initialize the session store.

```
...

import { Pool, PoolOptions } from 'pg';
import { SessionStore } from 'session_store.js';

...

//make sure you have a Pool setup
//with the appropriate options:
const poolOptions: PoolOptions = {
    host: 'localhost',
    user: 'your_user',
    port: 5432, //the port your database is running on, this is the default
    password: 'your_password',
    database: 'sessions',
    max: 20, //https://node-postgres.com/guides/pool-sizing
    maxUses: 0,
    idleTimeoutMillis: 30000,
    connectionTimeoutMillis: 2000,
    maxLifetimeSeconds: 60,
    allowExitOnIdle: false
};

const pool = new Pool(poolOptions);

const sessionStore = new SessionStore(pool);

app.use(session({
    secret: 'YOUR_SECRET',
    resave: false,
    saveUninitialized: false,
    store: sessionStore
}));
```

## Dependencies
For use with express, express-session and pg (postgres). See session_store.ts for details.

## Special Note

The session pruning feature that ships with connect-pg-simple is not implemented. The overall implementation is similar but has some differences. See [https://github.com/voxpelli/node-connect-pg-simple/](https://github.com/voxpelli/node-connect-pg-simple/) for details on how you might implement pruning if it is something you need. See [pg.Client](https://node-postgres.com/apis/client) for more information on available Pool options.
