---
title: "Express Proj Setup 5: Testing with Jest and Supertest"
date: "2022-02-02"
categories: 
  - "tech"
tags: 
  - "jest"
  - "mongodb"
  - "supertest"
  - "testing"
image: "/assets/blog/2-1.png"
---

Zero to complete, a step-by-step guide to setting up an Express project from a vet

This is part of an ongoing series of articles for setting up a full client/server Express based node application.

- 0: Environment Setup, Eslint, Airbnb style guide, Prettier
- 1: Project Configuration, Node env vs Project env
- 2: Middleware, Logging: Winston & Morgan
- 3: Data Storage: Native Mongodb
- 4: Authentication: Middleware & Secure Cookies
- **5: Testing: Jest and Supertest**
- 6: Code Documentation: JsDoc
- 7: Client JS bundling, Rollup
- 8: Security: Helmet
- 9: Optimization: Compression

Code is available at [github](https://github.com/paultman/full-express-setup) and is tagged at each stage of development.

## Testing

Eventually your app with grow to the point that manual testing is no longer tenable. As changes are made, you need a quick way to test the that your new code doesn’t adversely affect old code. That’s where automated testing comes in and we will be using a popular framework made by Facebook called [Jest](https://jestjs.io), and [Supertest](https://github.com/visionmedia/supertest) to mock network requests. We will also mock our database via a Jest [preset](https://github.com/shelfio/jest-mongodb) for an memory mongodb [package](https://github.com/nodkz/mongodb-memory-server).

We will start by adding them to our project as dependencies. Since they are not needed for a production deployment, we will use the -D flag to add as a dev dependancies.

npm i jest supertest @shelf/jest-mongodb  -D

When we started the project, we wisely created our app.js separate from our server.js which will make it easier to test.

Rather than a separate test directory, i prefer to have test files next to the files they are testing. That makes it easier for paths, maintenance, and being as close to the env of the file being tested. So lets start with creating app.test.js in our root directory. All our tests will have a similar naming structure: \[module\].test.js

// app.test.js
const request = require('supertest');
const { MongoClient } = require('mongodb');
const appConfig = require('./config/app');
const logger = require('./lib/logger');
const app = require('./app');

describe('App Routes', () => {
  let connection;
  let db;

  beforeAll(async () => {
    connection = await MongoClient.connect(process.env.MONGO\_URL, {
      useNewUrlParser: true,
      useUnifiedTopology: true,
    });
    db = await connection.db();
    app.init(appConfig, logger, db);
  });

  afterAll(async () => {
    await connection.close();
  });

  test('Should respond with a 302 to index.html for a non-authenticated user', async () => {
    const response = await request(app).get('/home.html');
    expect(response.status).toEqual(302);
  });

  test('POST to /register should respond with a cookie and redirect to protected home route', async () => {
    const data = { email: 'test@gmail.com', password: 'test\_Password' };
    await request(app)
      .post('/register')
      .send(data)
      .expect(302)
      .expect('Location', /home.html$/);
  });

  test('POST to /login with proper credentials should redirect to protected home route', async () => {
    const data = { email: 'test@gmail.com', password: 'test\_Password' };
    await request(app)
      .post('/login')
      .send(data)
      .expect(302)
      .expect('Location', /home.html$/);
  });

  test('GET to /logout should eliminate auth state cookie and be unable to go to protected route', async () => {
    await request(app).get('/logout').expect(200);
    await request(app).get('/home.html').expect(302);
  });
});

What do we have here? Well first, to mock requests, we use SuperTest. It gives us a request object and additional testing functions off of it. Since we added the database preset to Jest, all we need is the connect variable that our in-memory mongodb instance provides to MongoClient. It looks just like a normal mongodb database, evening using the same interface.

We import our app like normal, and test each of the routes we created earlier.

To run it, add a test script to your package.json

...
  "scripts": {
    "start:dev": "NODE\_ENV=development APP\_ENV=dev node server.js",
    "start:prod": "NODE\_ENV=production APP\_ENV=prod node server.js",
    "test:app": "NODE\_ENV=production APP\_ENV=test jest app.test.js"
  },
...

when being run, all our 5 tests should pass:

![](images/ecaf8-screen-shot-2022-02-02-at-12.36.55-pm.png)

You can find the current tag of v1.5, testing on github. Here's a comparison of [changes](https://github.com/paultman/full-express-setup/compare/v1.4...v1.5) since the last post focused on v1.4.

## References

Good collection of testing best practices:  
[https://github.com/goldbergyoni/javascript-testing-best-practices](https://github.com/goldbergyoni/javascript-testing-best-practices)
