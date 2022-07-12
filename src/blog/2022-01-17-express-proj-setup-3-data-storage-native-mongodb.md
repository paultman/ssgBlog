---
title: "Express Proj Setup: 3 Data Storage, Native Mongodb"
date: "2022-01-17"
categories: 
  - "tech"
tags: 
  - "express"
  - "mongodb"
image: "JSDoc-for-Automatic-Code-Documentation-2-1.png"
---

Zero to complete, a step-by-step guide to setting up an Express project from a vet

This is part of an ongoing series of articles for setting up a full client/server Express based node application.

- 0: Environment Setup, Eslint, Airbnb style guide, Prettier
- 1: Project Configuration, Node env vs Project env
- 2: Middleware, Logging: Winston & Morgan
- **3: Data Storage: Native Mongodb**
- 4: Authentication: Middleware & Secure Cookies
- 5: Testing: Jest and Supertest
- 6: Code Documentation: JsDoc
- 7: Client JS bundling, Rollup
- 8: Security: Helmet
- 9: Optimization: Compression

Code is available at [github](https://github.com/paultman/full-express-setup) and is tagged at each stage of development.

## ****Data Storage: Native Mongodb****

Now that we have a way to store project configuration, lets setup something that will be using it, our database. Since the move to javascript on the server-side via Nodejs, another trend has been the move away from formal schema based SQL based databases to dynamic, document based data stores which follow the familiar JSON structure, which client side developers are accustomed to.

You can find many articles comparing the differences, however for this one, it's assumed you want to use a document-based db, and choose one of the most popular ones, Mongodb.

First Install mongodb, you can find the instructions on their [webpage](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/). In my case, i’m using an M1 mac, and already have x-code command line tools installed, so I’ll using tap and install with brew.

\> brew tap mongodb/brew
> brew install mongodb-community@5.0

As per the docs, i’m going run it as a service, and since i’m a lazy programmer, I’ll make an alias in my .zshrc:

alias mongoDBStart='brew services start mongodb-community@5.0'
alias mongoDBStop='brew services stop mongodb-community@5.0'

Therefore, after running “mongoDBStart”, you can check the status with “brew services list”

Now to install the native mongo driver for our express template. If you’re wondering why I’m not using an ODM like Mongoose, there are many reasons for this. First, mongodb is already document oriented so working with it is easy, second it’s less performant, and last, I have more control over exactly what’s happening using the native driver. If you’re still not convinced, have a look at this reddit [discussion](https://www.reddit.com/r/node/comments/b1k1nt/mongodb_with_or_without_mongoose/ein4kju/?utm_source=reddit&utm_medium=web2x&context=3) from two years ago.

Ok, back to installing the native mongodb [driver](https://docs.mongodb.com/drivers/node/current/):

\> npm install mongodb

With MongoDB installed locally, and our native mongodb driver installed to our project we will integrated it into our project. Currently we start our server script which calls our express app to start our web server. Since our webserver will depend on a connected database, we will provision that first, and pass it when we initialize our express app. We will also add configuration settings to be read when setting up our db connection.

Rather than putting all our database code in its own file, likely in the lib directory, we will just directly put the code in server.js for simplicity.

// server.js
const { MongoClient } = require('mongodb');
const appConfig = require('./config/app');
const logger = require('./lib/logger');
const app = require('./app');

const localMongoURI = \`mongodb://${appConfig.db.host}:${appConfig.db.port}/?maxPoolSize=20&w=majority\`;
const mongoClient = new MongoClient(localMongoURI);
const db = mongoClient.db(appConfig.db.name);

(async function start() {
  app.init(appConfig, logger, db);
  app.listen(appConfig.app.port, () => {
    logger.info(\`Server listening at http://localhost:${appConfig.app.port}\`);
  });
})();

You can see we import the mongodb driver we just installed. We then setup the url we will be using to connect, create a db client, and select the database as defined in our application file.

Currently we do not do anything with our database, but in the next step we will add a users controller and add code to register new users and allow existing ones to login.
