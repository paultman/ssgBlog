---
title: "How to setup a complete Expressjs project"
date: "2021-11-04"
categories: 
  - "tech"
tags: 
  - "expressjs"
  - "jest"
  - "mongodb"
  - "morgan"
  - "vanillajs"
  - "webapp"
image: "b7848-express-middlewares-complete-guide.png.png"
---

In this guide, I'll show you how to setup a node web framework and use it to support a client webapp built with vanilla-javascript. It is not meant to teach you how to use Express itself, but after you've learned that, how best to set it up.

I've built many express based apps in the past and want to put together a list of current best practices as well as best resources/libraries as of Fall 2021. Packages in the NPM space change often, so I'm sure I'll need to keep updating this guide.

I'll start with what you will learn after reading this article:

- How to setup Visual Studio Code with a popular style guide so your code will have consistent style/formatting
- How to use Rollup to bundle code written in CJS/ESM formats
- Learn a common logging setup for an express app

## Logging, going beyond console.log

Having used many loggers for Express, the current go-to is to use [Winston](https://github.com/winstonjs/winston), in combination with Morgan.

We all know why logging is important, but I'll reiterate a few key points that Winston provides here.

- Separate information by type or log level, and filter which level is being captured for example depending on environment (production, development, etc)
- Can make logs more readable by adding color and custom formatting options
- Has transports to send log data to various sources besides the console, like to a file, database, or even stream to a log service.

Lets get started by installing Winston:

```
npm install winston
```

Here is their boilerplate configuration to create a Winston Logger, I've added more comments and a note below. Save it to ./lib/logger.js

const winston = require('winston');

const logger = winston.createLogger({ 
  level: 'info', // we want to pay attention to info level and below 
  format: winston.format.json(), // a good format, albeit without colors
  defaultMeta: { service: 'user-service' }, // extra data added to the log
  transports: \[ // here we define where logs should be sent, according to log level
    // - Write all logs with level \`error\` and below to \`error.log\`
    // - Write all logs with level \`info\` and below to \`combined.log\`
    new winston.transports.File({ filename: './log/error.log', level: 'error' }),
    new winston.transports.File({ filename: './log/combined.log' }),
  \],
});

// If we're not in production then log to the \`console\` with the format:
// \`${info.level}: ${info.message} JSON.stringify({ ...rest }) \`
if (process.env.NODE\_ENV === 'development') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple(),
  }));
}

I updated the end condition from !== 'production' to ==='development' as i'd rather not log those messages to console if the app is running in production and the NODE\_ENV wasn't set. In development if I don't see console messages, I can easily check if I've set my ENV correctly.

In the above example, you'll see we log all messages (info and below) to combined.log, and error messages to error.log. If we are in development, we also log everything to the console as well.

Let's update our server file to use Winston. In our callback from starting the server, change the message to use our new logger

// server.js
const express = require('express');
const path = require('path');
const config = require('./config');
const logger = require('./lib/logger');

const app = express();

app.use(express.static('dist'));
app.get('/', (req, res) =&gt; {
  res.sendFile(path.join(\_\_dirname, '/dist/index.html'));
});

app.listen(config.app.port, () =&gt; {
  logger.info(\`Server listening at http://localhost:${config.app.port}\`);
});

You will see a ./log directory created and in it, a combined.log file which will have all our messages.

Winston is great for logging our own custom messages, however you might also want to automatically log web requests. You can do that with an express middleware called [Morgan](https://expressjs.com/en/resources/middleware/morgan.html). Normally Morgan writes request data to standard output, the console, however you can also configure it to send request data to Winston.

To use it, first do the npm install:

```
npm install morgan
```

Import it into your express server code (1), use it with "dev" message format (2), and configure it (3)to use the Winston logger instance(4).

// server.js
const express = require('express');
const path = require('path');
const morgan = require('morgan'); //1
const config = require('./config');
const logger = require('./lib/logger');

const app = express();

app.use( //2, use it as a middleware for all requests
  morgan('combined', { //2 pass the middleware with log type and config
    stream: { //3 configure how to stream the request messages
      write: (message, encoding) =&gt; logger.info(message), //4, pass messages to our Logger
    },
  })
);
...

You might notice that we send all messages as "info" level to Winston. That's what we want because remember these are generic request information. Debugging, errors, etc we will handle with custom messages. These automatic ones should be of the "info" type.

There are various message formats, for example here are requests to "/" for our webserver:

- **combined**: {"level":"info","message":"::1 - - \[08/Oct/2021:13:20:28 +0000\] \\"GET / HTTP/1.1\\" 200 397 \\"-\\" \\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10\_15\_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.0 Safari/605.1.15\\"\\n"}
- **dev**: {"level":"info","message":"\\u001b\[0mGET / \\u001b\[32m200\\u001b\[0m 12.573 ms - 397\\u001b\[0m\\n"}
- **tiny**: {"level":"info","message":"GET / 200 397 - 1.981 ms\\n"}

Typically I'd keep Morgan request logs at "tiny" for normal usage and up the detail when doing investigation or for generating metrics with a 3rd party service like: solarwinds, datadog, or splunk.

## Generation Code Documentation

Any large project, and certainly one with multiple developers benefits by having good documentation, and the best part is that after adding comments to our code, we can automatically generate the documentation. We will do this both for our express server-side code, as well as our client-side js libraries.

We will use the popular [JSDoc](https://jsdoc.app) library and a utility to clean directories, which you can install via:

```
npm install jsdoc rimraf
```

We also need to configure how we will be using JSDoc, so create a configuration file in either .JS or .JSON format. Have a look at the JSDoc [config](https://jsdoc.app/about-configuring-jsdoc.html) page for details. I will use a basic js file, and export my configuration:

module.exports = {
  tags: {
    allowUnknownTags: true,
    dictionaries: \['jsdoc', 'closure'\],
  },
  source: {
    include: \['controllers'\],
    includePattern: '.+\\\\.js(doc|x)?$',
  },
  opts: {
    destination: 'docs/',
    recurse: true,
    verbose: true,
  },
};

The important parts of the file are telling JSDoc: what type of tags/annotations we will be using (you'll see this in the next section), where our files are located and what extensions they will have, and last, where we want to output our generated documentation files.

With that setup, it's time to add comments in our code which follow the JSDoc syntax. For this example, I will implement a popular api related to managing todo's, [TodoMVC](https://todomvc.com).

Here is a simple implementation where I define a JSON object of 3 tasks in and have two getter methods, one for a specific task, and another for all tasks. Save the file in a controllers directory as task.js

/\*\*
 \* @module task
 \*/

const tasks = {
  1: { descr: 'call Nick for upcoming birthday', state: 'open' },
  2: { descr: 'check travel requirements for Chile', state: 'open' },
  3: { descr: 'finish TPS report', state: 'done' },
};

/\*\*
 \* @public
 \* @function getTask
 \* @name GET /tasks/:id
 \* @description Takes a taskID from the path of the request and returns a task of the same ID
 \* @param {Object} req Express request with :id in path
 \* @param {Object} res call json method on request to return requested task in JSON format
 \* @returns {undefined}
 \*/
module.exports.getTask = function getTask(req, res) {
  res.json(tasks\[req.params.id\]);
};

/\*\*
 \* Get all tasks in database, regardless of state or ID
 \* @name GET /tasks/all
 \* @param {Object} req Express request
 \* @param {Object} res call json method on request to return all tasks in JSON format
 \* @returns {undefined}
 \*/
module.exports.getAllTasks = (req, res) =&gt; res.json(tasks);

For JSDoc to understand the code: First, I will annotate the file as a module, then for each function I annotate relative details like public private, the construct type, description, parameters, and the return type. You will follow a similar template for your own functions and can find all of them in the Block Tags section of the JSDoc [webpage](https://jsdoc.app/index.html)

With the JSDoc module installed, configured, and now implemented in our JS file, we can generate our docs.

In your package.json file, add the script action to clean the doc directory and invoke JSDoc:

```
"docs": "rimraf docs && jsdoc -c jsdoc.config.js"
```

Assuming no errors, you will see a docs directory created which contains files for a webpage which has documentation for your code.

### Bonus

If you are like me and would like a better looking layout then the default JSDoc one, you can use a custom template which defines alternate layouts. There are a few [here](https://cancerberosgx.github.io/jsdoc-templates-demo/demo/), personally I typically use [docdash](https://github.com/clenemt/docdash) which has lots of github stars and is regularly updated.

```
npm install docdash
```

In your configuration, add the line: "template": "node\_modules/docdash", to the ops section:

```
  opts: {
    template: 'node_modules/docdash',
    destination: 'docs/',
    recurse: true,
    verbose: true,
  },
```

Run the "npm run docs" command again and look at the docs generated. You'll see a layout with a left menu of your modules.

## Testing

Eventually your app with grow to the point that manual testing is no longer tenable. As changes are made, you need a quick way to test the that your new code doesn't adversely affect old code. That's where automated testing comes in and we will be using a popular framework made by Facebook called [Jest](https://jestjs.io), and [Supertest](https://github.com/visionmedia/supertest) to mock network requests.

We will start by adding them to our project as dev dependencies:

```
npm i jest supertest -D
```

Also, since jest will run its own server, we want to separate our express app code from server.js, so that we can require it in out test setup, and also in our production setup for server.js.

We will still kick off our server from server.js when not running tests, while tests will start from ./tests/app.test.js. Here's is the updated split between server.js and our newly created app.js:

// server.js
const config = require('./server.config');
const logger = require('./lib/logger');

const app = require('./app')(config, logger);

app.listen(config.app.port, () => {
  logger.info(\`Server listening at http://localhost:${config.app.port}\`);
});

// app.js
const morgan = require('morgan');
const path = require('path');
const express = require('express');
const taskRoutes = require('./controllers/task');

module.exports = (config, logger) => {
  const app = express();
  app.use(
    morgan('tiny', {
      stream: {
        write: (message) => logger.info(message),
      },
    })
  );
  app.use(express.static('dist'));

  app.get('/', (req, res) => {
    res.sendFile(path.join(\_\_dirname, '/dist/index.html'));
  });

  app.get('/task/all', taskRoutes.getAllTasks);
  app.get('/task/:id', taskRoutes.getTask);

  return app;
};

Our main entrypoint, server.js became very basic, simply pulling configuration, setting up the winston logger, initialize the app, and finally call .listen to start the express node server.

App.js on the otherhand gets all the express related middlewares and the configurtion and logging passed from server.js By splitting them up, we can now make app.test.js which will specifically require the newly created app.js

// app.test.js
const request = require('supertest');
const config = require('../server.config');
const logger = require('../lib/logger');

const app = require('../app')(config, logger);

describe('GET /task', () => {
  test('should respond with task for id 1 in JSON format', async () => {
    const response = await request(app).get('/task/1');
    expect(response.body.state).toBe('open');
  });

  test('should respond with an error when requesting an invalid task', async () => {
    const response = await request(app).get('/task/-1');
    expect(response.statusCode).toBe(500);
  });
});

We import supertest which we will use as a mock request object and have two tests for GET /task. The first tests getting a specific task, namely the task with ID of 1, and the second is a negative test to ensure that out app throws an error if an invalid id is called.

## Data Storage

Now to store our data and make it more interactive. Previously we used a hard coded JSON object to hold our tasks, now we will store tasks in a mongodb, non-sq, document based database.

First Install mongodb, you can find the instructions on their [webpage](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/). In my case, i'm using an M1 mac, and already have x-code command line tools installed, so I'll using tap and install.

```
> brew tap mongodb/brew
> brew install mongodb-community@5.0
```

As per the docs, i'm going run it as a service, and since i'm a lazy programmer, I'll make an alias in my .zshrc:

```
alias mongoDBStart='brew services start mongodb-community@5.0'
alias mongoDBStop='brew services stop mongodb-community@5.0'
```

Therefore, after running "mongoDBStart", you can check the status with "brew services list"

Now to install the native mongo driver for our express template. If you're wondering why I'm not using an ODM like Mongoose, there are many reasons for this. First, mongodb is already document oriented so working with it is easy, second it's less performant, and last, I have more control over exactly what's happening using the native driver. If you're still not convinced, have a look at this reddit [discussion](https://www.reddit.com/r/node/comments/b1k1nt/mongodb_with_or_without_mongoose/ein4kju/?utm_source=reddit&utm_medium=web2x&context=3) from two years ago.

Ok, back to installing the native mongodb [driver](https://docs.mongodb.com/drivers/node/current/):

```
> npm install mongodb
```

Now that we have Mongodb, we will work to create a database, create a collection, populate the newly created collection with our JSON starter data, and later build apis for CRUD operations.

### Database Migrations

Having done database migrations in the past, I typically like to do them with simple js scripts run by the node interpreter. So, for our starter project, I'll create a general "scripts" directory at the root of the project and database specific file called dbSeeding.js, this can also serve as a template for an actually db migration in the future.

const { MongoClient } = require('mongodb');
const config = require('../server.config');

MongoClient.connect(config.db.localMongoURI, (err, client) => {
  if (err) {
    console.log(\`have error: ${err}\`);
    return;
  }
  const db = client.db(client.s.databaseName);
  const starterCol = db.collection('ESTcol');
  const starterTasks = {
    1: { descr: 'call Nick for upcoming birthday', state: 'open' },
    2: { descr: 'check travel requirements for Chile', state: 'open' },
    3: { descr: 'finish TPS report', state: 'done' },
  };
  starterCol.insertOne(starterTasks, (err, doc) => {
    if (err) {
      console.log(err);
      throw err;
    } else console.log('Records inserted to db');
  });
});

Because this will be a manually run, non-production script, I'm going to be using the simple console to log status/errors.

You'll notice that we: imported database configuration, connected to our database, created/opened our collection an inserted our JSON data a moved our starter data there.

With our local database started, we will run our script through the node interpreter:

```
node dbOps.js
Records inserted to db {
  acknowledged: true,
  insertedId: new ObjectId("616c7d31f3b555051b39ea1e")
```

As we console logged the record in the callback, you can see that a we successfully created a new record which has the id of 616c7d31f3b555051b39ea1e (in string format). The id is natively in the database in BSON format.

### CRUD Ops

Now that we've got data in our database, time to write CRUD operations to interact with it. CRUD is a common acronym which stands for Create, Read, Update, Delete which represent that operations we will do against our db.

We will need to update our client code via HTML and Javascript updates. Here is our basic html page with a few styles included. It's pretty bare since most of the content will be added later via javascript.

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      #tasks {
        list-style-type: none;
      }
      input:checked + span.descr {
        text-decoration-line: line-through;
      }
    </style>
  </head>
  <body>
    <h1>Express-Starter-Template</h1>
    <form id="todoForm">
      <input id="newTask" />
      <button id="addTask">Add Task</button>
      <ul id="tasks"></ul>
    </form>
    <script type="text/javascript" src="./scripts/index.min.js"></script>
  </body>
</html>

As you can see, I have a few styles in the head of my document, a simple form, and I reference my Javascript at the end. As this is not an html/css best practice example, but rather focusing on CRUD db operations, I am not specific in my styles, and I litter my html with ID's :p

But enough of that, how about the referenced JS:

/\* index.js
  A simple client script to illustrate CRUD operations to an express server using mongodb
\*/
let nextSeqID = 0;

(function renderTasks() {
  const todoForm = document.getElementById('todoForm');
  const taskList = document.getElementById('tasks');

  // READ and render tasks
  fetch('/task/all')
    .then((resp) => resp.json())
    .then((data) => {
      let tasksStr = '';
      data.forEach((task) => {
        tasksStr += \`<li class='task' data-id='${task.\_id}'>
          <input type='checkbox' class='state' ${task.state === 'done' ? 'checked' : ''}>
          </input><span class='descr'>${task.descr}</span>
          <button class='delete'>delete</button></li>\`;
        nextSeqID = Math.max(nextSeqID, task.\_id);
      });
      taskList.innerHTML = tasksStr;
    });

  todoForm.addEventListener('click', (event) => {
    const { target } = event;
    const currID = target.parentNode.dataset.id;

    //  CREATE new TASK
    if (target.matches('#addTask')) {
      const newTaskInput = document.getElementById('newTask');
      const newTask = {
        \_id: ++nextSeqID,
        descr: newTaskInput.value,
        state: 'open',
      };
      fetch('/task', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(newTask),
      })
        .then((response) => response.json())
        .then((res) => {
          newTaskInput.value = '';
          console.log('Success:', res);
        })
        .catch((error) => {
          console.error('Error:', error);
        });
      // MODIFY a task by updating its state
    } else if (target.matches('input.state')) {
      const data = {
        state: target.checked ? 'done' : 'open',
      };
      fetch(\`/task/${currID}\`, {
        method: 'POST', // or 'PUT'
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
      })
        .then((response) => response.json())
        .then((res) => {
          console.log('Success:', res);
        })
        .catch((error) => {
          console.error('Error:', error);
        });
      // DELETE a certain task
    } else if (target.matches('button.delete')) {
      fetch(\`/task/${currID}\`, {
        method: 'DELETE',
      })
        .then((response) => response.json())
        .then((res) => {
          console.log('Success:', res);
        })
        .catch((error) => {
          console.error('Error:', error);
        });
    }
  });
})();

We have the code wrapped in an [IIFE](https://developer.mozilla.org/en-US/docs/Glossary/IIFE). First we make a network call to the server to get current tasks. Once we have them we build a list of tasks, each with a checkbox to represent state, and a delete button.

We also add a listener to our main form which, via [event bubbling](https://javascript.info/bubbling-and-capturing), we can target specific child actions.

The first is via a text input and button to add a new todo list item. Because mongodb doesn't have a built in way to increment simple numeric id's, we will do it from the client. In a production environment you should consider something more unique like a [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier). This would could also be taken care of by MongoDB which uses [BSON](https://en.wikipedia.org/wiki/BSON) encoded record id's, but to keep things simple, we will use simple integer Id's. For this reason, you'll see we track the variable called nextSeqID which is set to the highest ID we notice in our initial task render, and is incremented as we add tasks. With our record, we use Fetch to post it to our web server and eventually refresh our page to see the newly add record.

We make similar actions for modifying an item, ie when we watch to change a task state from open to done or vice versa, and when we want to delete an item. We call fetch with a certain path and method type which matches a signature we have in our express routes.

// app.js

...

  app.use(express.json());
  app.use(express.urlencoded({ extended: true }));

  app.get('/task/all', taskRoutes.getAll);
  app.get('/task/:id', taskRoutes.get);
  app.post('/task', taskRoutes.create);
  app.post('/task/:id', taskRoutes.update);
  app.delete('/task/:id', taskRoutes.delete);
};

module.exports = app;

As you can see, we call methods like get, post and delete, pass them url path signatures, and a corresponding controller method.

/\*\*
 \* @module task
 \*/

module.exports.init = async (config, logger, db) => {
  this.tasksCol = db.collection('tasks');
};

/\*\*
 \* @public
 \* @function getTask
 \* @name GET /tasks/:id
 \* @description Takes a taskID from the path of the request and returns a task of the same ID
 \* @param {Object} req Express request with :id in path
 \* @param {Object} res Call json method on request to return requested task in JSON format
 \* @returns {undefined}
 \*/
module.exports.get = (req, res) => {
  const taskID = parseInt(req.params.id, 10);
  this.tasksCol
    .findOne({ \_id: taskID })
    .then((record) => {
      if (record) res.json(record);
      else res.json({});
    })
    .catch((err) => res.json(err));
};

/\*\*
 \* Get all tasks in database, regardless of state or ID
 \* @name GET /tasks/all
 \* @param {Object} req Express request
 \* @param {Object} res Call json method on request to return all tasks in JSON format
 \* @returns {undefined}
 \*/
module.exports.getAll = (req, res) => {
  this.tasksCol
    .find({})
    .toArray()
    .then((results) => res.json(results))
    .catch((err) => res.json(err));
};

/\*\*
 \* @public
 \* @function create
 \* @name POST /tasks
 \* @description Takes a JSON object and creates a new task
 \* @param {Object} req Express request with a JSON body
 \* @param {Object} res Return an empty object on success, else error message
 \* @returns {undefined}
 \*/
module.exports.create = (req, res) => {
  const newTask = req.body;
  this.tasksCol
    .insertOne(newTask)
    .then((record) => {
      if (record) res.json(record);
      else res.json({});
    })
    .catch((err) => res.json(err));
};

/\*\*
 \* @public
 \* @function update
 \* @name POST /tasks/:id
 \* @description Takes a taskID from the path of the request and updates record passed on req body
 \* @param {Object} req Express request with task to be updated and updated field
 \* @param {Object} res Return an empty object on success, else error message
 \* @returns {undefined}
 \*/
module.exports.update = (req, res) => {
  const taskID = parseInt(req.params.id, 10);
  this.tasksCol
    .updateOne({ \_id: taskID }, { $set: { state: req.body.state } })
    .then((record) => {
      if (record) res.json(record);
      else res.json({});
    })
    .catch((err) => res.json(err));
};

/\*\*
 \* @public
 \* @function delete
 \* @name DELETE /tasks/:id
 \* @description Takes a taskID from the path of the request and deletes that record
 \* @param {Object} req Express request with :id in path
 \* @param {Object} res Call json method on request to return requested task in JSON format
 \* @returns {undefined}
 \*/
module.exports.delete = (req, res) => {
  const taskID = parseInt(req.params.id, 10);
  this.tasksCol
    .deleteOne({ \_id: taskID })
    .then((result) => {
      res.json({ result: \`${result.deletedCount} deleted\` });
    })
    .catch((err) => res.json(err));
};

As you can see, we initialize our module with our database collection, and for each of our CRUD methods, directly invoke operations on our database. The db operations are asynchronous and return promises, so we use the .then/.catch syntax.

## Updated Tests

Now that we have more logic and full CRUD functionality, we can update our Jest tests to ensure our app will continue to work properly. Rather then using our actual database, we will use an in-memory Mongodb. The easiest, and [recommended](https://jestjs.io/docs/mongodb) way, is to use the Jest Mongodb preset which installs the in-memory Mongodb module and sets up Jest to use it. You can find the package and readme on the [github](https://github.com/shelfio/jest-mongodb) page.

Once it is installed, and working with Jest, we will update our app.test.js file like so:

// app.test.js
const request = require('supertest');
const { MongoClient } = require('mongodb');
const config = require('../server.config');
const logger = require('../lib/logger');

const app = require('../app');

let connection;
let db;

beforeAll(async () => {
  connection = await MongoClient.connect(global.\_\_MONGO\_URI\_\_, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });
  db = await connection.db();
  app.init(config, logger, db);
});

afterAll(async () => {
  await connection.close();
});

describe('POST /task', () => {
  test('should create a new task', async () => {
    const response = await request(app).post('/task').send({ \_id: 1, desr: 'task One', state: 'open' });
    expect(response.statusCode).toBe(200);
  });

  test('should respond with task for id 1 in JSON format', async () => {
    const response = await request(app).get('/task/1');
    expect(response.body.state).toBe('open');
  });

  test('should respond with an updated record after changing a field', async () => {
    const response = await request(app).post('/task/1').send({ state: 'done' });
    expect(response.body.modifiedCount).toBe(1);
  });

  test('should respond with an a message of 1 record  deleted', async () => {
    const response = await request(app).delete('/task/1');
    expect(response.body.result).toBe('1 deleted');
  });

  test('should respond with an empty record when requesting an invalid task', async () => {
    const response = await request(app).get('/task/1');
    expect(response.body).toStrictEqual({});
  });
});

As you can see, we bring in the normal MongoClient, setup our database in the beforeAll section of our code and pass that database to our express app. Our routes will then use that database and we can tech each of our task apis.

## Authentication

We will start with creating a local, basic email/password component where we will handle authentication ourselves. We will need:

- Some way to persist login state on the client
- A middleware to check authenticated state on protected routes
- A registration page to create new users
- A login page to accept and check user credentials

Later we will also cover OAuth, and use a common library called Passport, to leverage 3rd party login systems like those from Google and Microsoft. Last we will also need to associate talks to the _current user rather than a single default user._

Implementation overview:

1. We will update our app.js to add routes, and to add middleware to check for state info from our client (via a cookie).
2. We need to update our tasks controller and associated tasks.html page, as well add a users controller.
3. We will update our landing page, index.html, to contain a form to allow for either registration or login. We will also add two convenience links to try to go to our protected "tasks.html" page, and another to logout our user if they have already authenticated.

Ok, lets start with the UI to register a user. Infact we can combine both the register and login UI. Remember we simply need something to exercise our expressjs code, so we aren't winning any UI awards here :p

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <p><a href="tasks.html">Go to protected route: tasks</a></p>
    <p><a href="/logout">Logout, delete client state token</a></p>

    <form action="/enter" method="post">
      Email: <input type="text" name="email" /> Password: <input type="text" name="password" />
      <br />
      <input type="radio" name="action" value="register" /> Register
      <input type="radio" name="action" value="login" checked="true" /> Login
      <button id="processForm" type="submit" disabled>submit</button>
    </form>
    <script type="text/javascript">
      document.forms\[0\].onchange = function () {
        let empty = false;
        document.forms\[0\].querySelectorAll('input\[type = "text"\]').forEach((elm) => {
          if (!empty && elm.value.trim() == '') empty = true;
        });
        document.getElementById('processForm').disabled = empty;
      };
    </script>
  </body>
</html>

Pretty basic, we have a radio button to toggle between registration and login. We have our email and password fields and submit button to post to '/enter'. We also added two convenience links: first to check for state cookie via going to a protected route, and another to "logout" and destroy the client state cookie.

Since we are using / for the login page, we create tasks.html and do task rendering there:

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      #tasks {
        list-style-type: none;
      }
      input:checked + span.descr {
        text-decoration-line: line-through;
      }
    </style>
  </head>
  <body>
    <h1>Express-Starter-Template</h1>
    <form id="todoForm">
      <input id="newTask" />
      <button id="addTask">Add Task</button>
      <ul id="tasks"></ul>
    </form>
    <script type="text/javascript" src="./scripts/tasks.min.js"></script>
  </body>
</html>

And the associated javascript file:

/\* tasks.js
  A simple client script to illustrate CRUD operations to an express server using mongodb
\*/
let nextSeqID = 0;

(function renderTasks() {
  const todoForm = document.getElementById('todoForm');
  const taskList = document.getElementById('tasks');

  // READ and render tasks
  fetch('/task/all')
    .then((resp) => resp.json())
    .then((data) => {
      let tasksStr = '';
      data.forEach((task) => {
        tasksStr += \`<li class='task' data-id='${task.\_id}'>
          <input type='checkbox' class='state' ${task.state === 'done' ? 'checked' : ''}>
          </input><span class='descr'>${task.descr}</span>
          <button class='delete'>delete</button></li>\`;
        nextSeqID = Math.max(nextSeqID, task.\_id);
      });
      taskList.innerHTML = tasksStr;
    });

  todoForm.addEventListener('click', (event) => {
    const { target } = event;
    const currID = target.parentNode.dataset.id;

    //  CREATE new TASK
    if (target.matches('#addTask')) {
      const newTaskInput = document.getElementById('newTask');
      const newTask = {
        \_id: ++nextSeqID,
        descr: newTaskInput.value,
        state: 'open',
      };
      fetch('/task', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(newTask),
      })
        .then((response) => response.json())
        .then((res) => {
          newTaskInput.value = '';
          console.log('Success:', res);
        })
        .catch((error) => {
          console.error('Error:', error);
        });
      // MODIFY a task by updating its state
    } else if (target.matches('input.state')) {
      const data = {
        state: target.checked ? 'done' : 'open',
      };
      fetch(\`/task/${currID}\`, {
        method: 'POST', // or 'PUT'
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
      })
        .then((response) => response.json())
        .then((res) => {
          console.log('Success:', res);
        })
        .catch((error) => {
          console.error('Error:', error);
        });
      // DELETE a certain task
    } else if (target.matches('button.delete')) {
      fetch(\`/task/${currID}\`, {
        method: 'DELETE',
      })
        .then((response) => response.json())
        .then((res) => {
          console.log('Success:', res);
        })
        .catch((error) => {
          console.error('Error:', error);
        });
    }
  });
})();

Now we need to add a cookie-session middleware to our app and setup routes to support the form post we created above.

// app.js
const morgan = require('morgan');
const path = require('path');
const express = require('express');
const cookieSession = require('cookie-session');
const taskRoutes = require('./controllers/tasks');
const userRoutes = require('./controllers/users');

const app = express();
app.locals.projDir = \_\_dirname;

app.use(
  cookieSession({
    name: 'session',
    maxAge: 1000 \* 60 \* 60 \* 24 \* 7, // miliseconds from now to expire, 1wk
    httpOnly: true, // unreadable via client JS
    sameSite: 'lax', // only sent from same website
    signed: true, // include signature to check tampering
    // secure: true, // must be over https
    keys: \['s3cr3t'\],
  })
);

const checkAuth = (req, res, next) => {
  if (!(req.session && req.session.ref)) {
    if (req.headers\['x-requested-with'\]) {
      // if request is an ajax request send json back
      res.status(302).send({ url: '/' });
    } else {
      res.redirect('/'); // do normal 302 redirect
    }
  } else next(); // they have an untampored with token, could further validatate here
};

app.init = (config, logger, db) => {
  taskRoutes.init(config, logger, db);
  userRoutes.init(config, logger, db);
  app.use(
    morgan('tiny', {
      stream: {
        write: (message) => logger.info(message),
      },
    })
  );

  app.use(express.json());
  app.use(express.urlencoded({ extended: true }));

  app.get('/task/all', taskRoutes.getAll);
  app.get('/task/:id', taskRoutes.get);
  app.post('/task', taskRoutes.create);
  app.post('/task/:id', taskRoutes.update);
  app.delete('/task/:id', taskRoutes.delete);
  app.post('/enter', userRoutes.enter);
  app.get('/tasks.html', checkAuth, (req, res) => {
    res.sendFile(path.join(\_\_dirname, '/dist/tasks.html'));
  });
  app.get('/', (req, res) => {
    if (req.session && req.session.user) res.sendFile(path.join(\_\_dirname, '/dist/tasks.html'));
    else res.sendFile(path.join(\_\_dirname, '/dist/index.html'));
  });
  app.get('/logout', userRoutes.logout);
  app.use(express.static('dist'));
};

module.exports = app;

Last we need to update our user controller to handle the enter route:

/\*\*
 \* @module users
 \*/

const bcrypt = require('bcrypt');
const path = require('path');

module.exports.init = async (config, logger, db) => {
  this.userCol = db.collection('users');
};

/\*\*
 \* @public
 \* @function enter
 \* @name POST /enter
 \* @description takes a form as input and either registeres a new user or saves user ref to session
 \* @param {Object} req Express request with form elements in body of post
 \* @param {Object} res return set cookie or error message
 \* @returns {undefined}
 \*/

module.exports.enter = async (req, res) => {
  if (req.body.action === 'register') {
    const user = {
      email: req.body.email,
      passHash: bcrypt.hashSync(req.body.password, bcrypt.genSaltSync(10)),
    };
    const ref = await this.userCol.insertOne(user);
    // if (!req.session) req.session = {};
    req.session.ref = ref.insertedId.toString();
    res.sendFile(path.join(req.app.locals.projDir, '/dist/tasks.html'));
  } else {
    const record = await this.userCol.findOne({ email: req.body.email });
    if (record === null) {
      res.status('401').json({ msg: \`no record found with email ${req.body.email}\` });
    } else if (!bcrypt.compareSync(req.body.password, record.passHash)) {
      res.status('403').json({ msg: 'wrong password' });
    } else {
      req.session.ref = record.\_id.toString();
      res.sendFile(path.join(req.app.locals.projDir, '/dist/tasks.html'));
    }
  }
};

/\*\*
 \* @public
 \* @function logout
 \* @name get /logout
 \* @description this function simply clears a httponly cookie from the client by setting it to empty string
 \* @param {Object} req Express request
 \* @param {Object} res empty body but cookie middleware sets response header
 \* @returns {undefined}
 \*/

module.exports.logout = async (req, res) => {
  req.session = null;
  res.sendFile(path.join(req.app.locals.projDir, '/dist/index.html'));
};

Gone are the days of a simple email login. Now we want to give our users more options, including using popular auth accounts they already have. Many providers have adopted [OAuth](https://en.wikipedia.org/wiki/OAuth), an open standard authorization protocol. Rather than reinvent the wheel, we will use a library called Passport JS, along with strategies for each provide we are interested in supporting. We will also use it to support our own email login via [JWTs](https://en.wikipedia.org/wiki/JSON_Web_Token).

HTTP itself is a stateless protocol. Normally, when you browse to a website, without sessions or cookies, the server doesn't know who you are or have any state information about you. Its job is to simply respond to requests. Normally these requests are anonymous, but a client can also want custom data, just for them. They can send extra data in their request with a kind of unique signature. With that, the server can know more about the request for example if you've been there before, and if you authenticate with the app, it can match your request with an identity.

There are two ways to do this:

The traditional way was for a website would add a cookie to your web browser with a unique identifier and on the server there would be a record which ties that unique identifier to a bunch of other data. This is called a server-side session. The benefit is that the web browser itself has no sensitive data, only the unique session identifier, however the problem is that the session data needs to be saved on the server and always available. If it's in memory, what if the server needs to be restarted? If it's saved to the database, that's a lot of extra work to make a call to a db for each request, add the session data, then go about the normal work of fulfilling the request.

An alternative is to eliminate the server state and keep session information on the client itself rather than just the server-side reference ID. By doing it this way, we don't have the drawbacks of keeping state information on the server (cache or database), and our api's can still process requests with state/user info. One concern with this method is security, rather than the server holding the user info, we have that data on the client.

We will secure our cookie via two tools, first: secure cookies to prevent client side updates (cookies will only be set and updated by the server), and second: use of [JSON Web](https://jwt.io) [Tokens](https://en.wikipedia.org/wiki/JSON_Web_Token). to do this, we will use Passport, a popular library that handles authentication and has a [JWT strategy](http://www.passportjs.org/packages/passport-jwt/). This is the manner we will walk though in this guide. We will also use passport later when we want to leverage login systems of other platforms..

at a high level we will:

1. Generate public and private key pairs which are specific to our server
2. When a user registers we will use these key value pairs, along with account information to generate a JWT which we will send to the client.
3. The client writes the JWT to a secure cookie and sends it back to the server on every future request.
4. On subsequent logins, a JWT is created and sent to the client, and used on future requests.
5. For each protected route, Passport middleware will validate the cookie/JWT and add user info to the request to be used by downstream middleware.

UPDATE:

In my research, I've decided to ditch using JWT's for local webapp authentication. Why?

- Many of the benefits I wanted via an authentication setup come by using a simple cookie, and one in particular, tampering detection comes when using express cookies.
- Because JWT's are generic, they include extra information which I don't need in my singular implementation. For example JWT's themselves start with a header to indicate the algorithm for the signature, and even the type, which is "JWT." The additional information which makes them portable, also makes them bigger.
- The JWT spec needs to be generic so it can be used in a variety of situations, so the payload is in JSON format which is easy to work with and supports nesting of data. However, in my case the data is flat. Simple key/value pair(s) are all that I need.
- Last, because my server is both generating and validating the token, I don't need asymmetric encryption of a private/public key pair. The client does not need to read the data (decrypt), possibly modify and encrypt the data. Only the server needs to perform those operations so only a single symmetric key is all I need. A single key is also [faster to sign and verify](https://sectigostore.com/blog/5-differences-between-symmetric-vs-asymmetric-encryption/).

Last, is that my preferred mechanism for storing and transferring it on the client is via cookies and that's not the most natural fit for JWT's. There are many JWT implementations for example: saving to local storage then manually setting an AUTH header in a fetch request, or sticking it in a post body or even a query parameter. There was even a Passport JWT strategy which levered cookies, but that project (still on the [Passport page](https://www.google.com/search?client=safari&rls=en&q=passport-jwt-cookie+combo&ie=UTF-8&oe=UTF-8)) has been [archived](https://github.com/codebarista/passport-jwt-cookiecombo) by the owner. So from a code style standpoint, it didn't feel as good as simply leveraging req.cookies from Express.

With the reasons explained, time to implement authenitication using signed cookie. Since we need signed cookies, we can leverage Expressjs [cookies](https://github.com/pillarjs/cookies). Besides checking for tampering (validation), another advantage of the cookies library is that verification/processing is done lazily, as needed. Rather than writing out own middleware, we will use a package called [cookie-session](https://github.com/expressjs/cookie-session), to protect our routes. It will check the validity of the cookie token, and add the cookie data to our request so that it's available to downstream middleware.

what does cookie session do more then just cookies

We start by adding a few modules:

```
npm i passport-jwt
```

deciding on JWT vs Signed Cookie

Evaluation and production implementation Example: https://bloggle.coggle.it/post/190706036692/what-weve-learned-from-moving-to-signed-cookies

Reach out for other opinions:  
[https://stackoverflow.com/questions/69945122/express-signed-cookie-vs-jwt-as-cookie-for-authentication](https://stackoverflow.com/questions/69945122/express-signed-cookie-vs-jwt-as-cookie-for-authentication)

NPM cookie-session:  
https://github.com/expressjs/cookie-session

Great Video on Authentication and how Passport works:  
"https://www.youtube.com/watch?v=F-sFp\_AvHc8&t=65s

http://expressjs.com/en/api.html#res.cookie  
  
node-jsonwebtoken  
https://github.com/auth0/node-jsonwebtoken

Express request.cookies  
http://expressjs.com/en/api.html#req.cookies

token+cookie:  
https://blog.bitsrc.io/why-using-tokens-and-cookies-together-is-better-for-web-apps-9d205b7c1961

anti JWT for sessions  
https://productioncoder.com/should-you-put-jwt-in-a-cookie-or-local-storage/  
http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/

passport jwt  
https://github.com/mikenicholson/passport-jwt

express setups:  
https://helabenkhalfallah.medium.com/nodejs-rest-api-with-express-passport-jwt-and-mongodb-98e5f2fee496  
https://dev.to/calvinqc/a-step-by-step-guide-to-setting-up-a-node-js-api-with-passport-jwt-5fa5  
https://www.zachgollwitzer.com/posts/2020/passport-js/  
https://javascript.plainenglish.io/jwt-auth-with-node-and-passport-js-c41a91d333e0  
https://alphonso-javier.medium.com/building-httponly-cookie-jwt-authentication-with-passport-js-27ec519b99c1  
  
express behind proxy: http://expressjs.com/en/guide/behind-proxies.html

## Conclusion

I hope you enjoyed this vanilla js walk-though. You can find the final version of code here, which has been tagged following every section.

We have created a fully functioning client app and express web server which leverages a mongodb. The app includes testing, document generation, and external configuration. This guide has gotten a lot longer than I imagined, so I'll most likely break it into multiple posts, each focusing on a specific section.

## References

Logging, good overview on using Winston: [https://www.youtube.com/watch?v=A5YiqaQbsyI](https://www.youtube.com/watch?v=A5YiqaQbsyI)

Strict mode still required? [https://tvernon.tech/blog/javascript-strict-mode](https://tvernon.tech/blog/javascript-strict-mode)

Performance/Optimization: [https://expressjs.com/en/advanced/best-practice-performance.html](https://expressjs.com/en/advanced/best-practice-performance.html)

Jest with Supertest for Express testing: [https://www.albertgao.xyz/2017/05/24/how-to-test-expressjs-with-jest-and-supertest/](https://www.albertgao.xyz/2017/05/24/how-to-test-expressjs-with-jest-and-supertest/)

intall local mongoDB via homebrew: [https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-os-x/)
