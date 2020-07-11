# Jira Clone with Nx

This project is generated with [Nx](https://nx.dev)

## Environment Setup

This project needs **MongoDB** and **Redis** running locally. One quick way of doing so is to use **Docker**. Pull the official images for both, setup your `docker port` and run both.

- MongoDB Port: 27017
- Redis Port: 6379

```bash
$ docker run -d -p 27017-27019:27017-27019 --name mongodb mongo:4.2
$ docker run -d -p 6379:6379 --name redis -d redis
```

You would also need **NodeJS** installed.

## Local Development

1. `npm install` to install all dependencies
2. Run your Docker containers
3. `npm run start:api` to run the API (written with [NestJS](https://nestjs.com))
4. `npm run start:ng` to run the Angular application (Upcoming)

### Migrations

1. Make sure to run the application once to have the collections created with the correct Schema + indexes
2. `cd mongo-migrations`
3. `MIGRATE_SYSTEM_ADMIN_PASSWORD='your_system_admin_password' npx migrate-mongo up`

### GraphQL Playground

Make sure to set `request.credentials` to `"include"` for **Cookie Authentication**.

## File Structure

Since the project is created by **Nx**, the file structure closely follows **Nx** structure.

```
ngvn (root)
|____apps
|____libs
|____mongo-migrations
|____package.json
|____angular.json
|____nx.json
|____schema.gql
|____...
```

#### Root

Root directory contains mostly configuration files for the project such as: `package.json`, `tsconfig.json`, `tslint.json`, `nx.json`, and `angular.json`. The two most important files that
you'd probably touch are `nx.json` and `angular.json`.

- `nx.json`: This is the **Nx Workspace** configuration files where it contains information about all the `libs` and `apps` in the workspace along with their internal dependencies (not npm packages)
- `angular.json`: Since this is an **Angular** + **Nest** application so of course there is an `angular.json`. If you're not familiar with **Angular**, `angular.json` is the configuration file for
  `Angular CLI`. In a **Nx Workspace**, `Angular CLI` (as well as `Nest CLI`) will be patched by the `Nx CLI`. So all your commands will be run with `nx` instead of `ng`.

There is also a `schema.gql` file which is generated by `@nestjs/graphql` based on the `Resolvers` defined in the `api`.

#### `apps`

This folder houses all current `apps` within the `workspace`. There are 3 apps at the moment:

- `api`: **Nest** application with **GraphQL** and **MongoDB**
- `jira-clone`: **Angular** application
- `jira-clone-e2e`: **Angular** E2E testing application powered by **Cypress**

#### `libs`

This folder houses all related `libs` to be used for `apps` (either by a single specific app or shared within apps). Three main top-level libs are:

- `api`: All `libs` to be used by the `api` app. This lib houses all **Feature Modules** with related **Models**, **Repositories**, **Services**, and **Resolvers**. It also has **Security** related
  stuffs like: **Auth** and **Permissions**.
- `background`: All `libs` to be used by the `api` application for **Background Tasks** (powered by [node-bull](https://optimalbits.github.io/bull/)
  and wrapped by [@nestjs/bull](https://github.com/nestjs/bull)).
- `shared`: All `libs` to be used by both the `api` and `jira-clone` applications.

#### `mongo-migrations`

This folder is to house migration scripts for the project's database. [migrate-mongo](https://www.npmjs.com/package/migrate-mongo) is the npm package used to power the migrations.

## Environment Variables

The project does use **Environment Variables** provided from `process.env`, however, we don't use `.env` files as we think it is not robust enough. Plus, we would have to parse the environment variable
as a whole.

There are 5 (as of the moment) main configurations whose values can be provided by `process.env`:

#### AppConfiguration

Main application configuration like `host`, `port`, and `env` etc.

| var name      | type   | default               | description                                                                                                                                                              |
| ------------- | ------ | --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| APP_HOST      | string | localhost             | The host of the application less the protocol                                                                                                                            |
| APP_PORT      | number | 8080                  | The port of the application                                                                                                                                              |
| APP_DOMAIN    | string | http://localhost:8080 | Full domain of the application                                                                                                                                           |
| NODE_ENV      | string | development           | The environment that the application is running on. Most of the time, the cloud platform that the application is deployed will set `NODE_ENV` to `production` by default |
| CLIENT_DOMAIN | string | http://localhost:4200 | The domain of the frontend application. This is used to setup `CORS` for Cookie Authentication as well as email related (upcoming if needed) operations                  |

#### ArenaConfiguration

The `api` uses `bull-arena` as a dashboard for the **Bull Queues** defined in the application. It is a nice interface to interact with the **Queues** such as: view jobs, retry failed jobs, manually queue new job etc.

| var name             | type    | default   | description                             |
| -------------------- | ------- | --------- | --------------------------------------- |
| ARENA_DISABLE_LISTEN | boolean | false     | Whether to turn on `bull-arena` or not  |
| ARENA_HOST           | string  | localhost | The host that `bull-arena` will run on. |
| ARENA_PORT           | number  | 8080      | The port that `bull-arena` will run on. |

#### AuthConfiguration

This configuration will determine how `nestjs/passport` and `nestjs/jwt` behaves.

| var name            | type   | default            | description                                                                           |
| ------------------- | ------ | ------------------ | ------------------------------------------------------------------------------------- |
| JWT_SECRET          | string | superSecret!       | The secret (for `accessToken`) that `jsonwebtoken` will use to sign the payload with. |
| JWT_EXPIRED         | string | 15m                | The expiration of `accessToken`                                                       |
| REFRESH_JWT_SECRET  | string | superSecretSecret! | The secret (for `refreshToken`) that `jsonwebtoken` will use to sign the payload with |
| REFRESH_JWT_EXPIRED | string | 7d                 | The expiration of `refreshToken`                                                      |
| JWT_SALT            | number | 12                 | Salt value for `bcrypt`                                                               |

#### DbConfiguration

**MongoDB** configuration

| var name               | type    | default                    | description                                                                          |
| ---------------------- | ------- | -------------------------- | ------------------------------------------------------------------------------------ |
| MONGO_URI              | string  | mongodb://localhost:27017/ | The URI that **MongoDB** will run on                                                 |
| MONGO_DB_NAME          | string  | jira-clone-local           | The name of the database                                                             |
| MONGO_RETRY_ATTEMPTS   | number  | 5                          | The number of times `@nestjs/mongoose` will try to connect to the database if failed |
| MONGO_RETRY_DELAY      | number  | 1000                       | The delay between retries                                                            |
| MONGO_FIND_AND_MODIFY  | boolean | false                      | `useFindAndModify` option                                                            |
| MONGO_NEW_URL_PARSER   | boolean | true                       | `useNewUrlParser` option                                                             |
| MONGO_CREATE_INDEX     | boolean | true                       | `useCreateIndex` option                                                              |
| MONGO_UNIFIED_TOPOLOGY | boolean | true                       | `useUnifiedTopology` option                                                          |

#### RedisConfiguration

**Redis** configuration that will power `bull-arena`, `nestjs/bull`, and `CachingService`

| var name            | type    | default   | description                                  |
| ------------------- | ------- | --------- | -------------------------------------------- |
| REDIS_CACHE_ENABLED | boolean | true      | Whether **Redis** should be enabled          |
| REDIS_HOST          | string  | localhost | The host that **Redis** will run on          |
| REDIS_PORT          | number  | 6379      | The port that **Redis** will run on          |
| REDIS_TTL           | number  | 86400     | Time-to-live configuration. Default to 1 day |

## Tech Stack

- [Nx](https://nx.dev)
- [Angular](https://angular.io)
- [NestJS](https://nestjs.com)
- [MongoDB](https://www.mongodb.com/)
- [Redis](https://redis.io/)