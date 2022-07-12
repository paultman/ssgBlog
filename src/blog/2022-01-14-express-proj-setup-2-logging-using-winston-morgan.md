---
title: "Express Proj Setup: 2 Logging, using Winston &amp; Morgan"
date: "2022-01-14"
categories: 
  - "tech"
tags: 
  - "logging"
  - "morgan"
  - "winston"
image: "JSDoc-for-Automatic-Code-Documentation-3.png"
---

Zero to complete, a step-by-step guide to setting up an Express project from a vet

This is part of an ongoing series of articles for setting up a full client/server Express based node application.

- 0: Environment Setup, Eslint, Airbnb style guide, Prettier
- 1: Project Configuration, Node env vs Project env
- **2: Middleware, Logging: Winston & Morgan**
- 3: Data Storage: Native Mongodb
- 4: Authentication: Middleware & Secure Cookies
- 5: Testing: Jest and Supertest
- 6: Code Documentation: JsDoc
- 7: Client JS bundling, Rollup
- 8: Security: Helmet
- 9: Optimization: Compression

Code is available at [github](https://github.com/paultman/full-express-setup) and is tagged at each stage of development.

## Middleware, Logging: Winston & Morgan

There comes a point when just using console.log just doesn't cut it. Having used many loggers for Express, the current go-to is to use [Winston](https://github.com/winstonjs/winston), in combination with Morgan.

We all know why logging is important, but I’ll reiterate a few key points that Winston provides here. 

- Separate information by type or log level, and filter which level is being captured for example depending on environment (production, development, etc)
- Can make logs more readable by adding color and custom formatting options
- Has transports to send log data to various sources besides the console, like to a file, database, or even stream to a log service.

With Winston doing all that, what's the use of Morgan? Morgan is an express [middleware](https://expressjs.com/en/guide/using-middleware.html) which automatically generates logging for http, web-server interactions. We will install it later.

Lets get started by installing Winston:

npm install winston

Here is their boilerplate configuration to create a Winston Logger, I’ve added more comments and a note below. Save it to ./lib/logger.js

// ./lib/logger.js
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
if (process.env.NODE\_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple(),
  }));
}
module.exports = logger;

In the above example, you’ll see we log all messages (info and below) to combined.log, and error messages to error.log. If we are not in production mode, we also log everything to the console as well. Btw, this NODE\_ENV setting goes back to the explanation we had in the last post where we talked about the danger double using this as also the app environment setting. Since we have explicit configurations, this will not be a problem when in 'staging' or 'testProduction' app environments.

Let’s update our server.js file to use the winston logger we created above. We will pass it as a param when initializing our express app.

// server.js
const appConfig = require('./config/app');
 const logger = require('./lib/logger');
const app = require('./app');

(async function start() {
  app.init(appConfig, logger);
  app.listen(appConfig.app.port, () => {
    logger.info(\`Server listening at http://localhost:${appConfig.app.port}\`);
  });
})();

We will update our app.js file later, but now you can run the app and you will see a ./log directory created and in it, a combined.log file with our server startup message.

Winston is great for logging our own custom messages, however you might also want to automatically log web requests. You can do that with [Morgan](https://expressjs.com/en/resources/middleware/morgan.html), the express middleware we mentioned earlier. Normally Morgan writes request data to standard output, the console, however we can configure it to use the Winston logger we configured earlier.

To use it, first do the npm install:

npm install morgan

Then update our app.js file. Import it into your express server code (1), use it with “dev” message format (2), and configure it (3)to use the Winston logger instance(4).

// app.js

const express = require('express');
const morgan = require('morgan'); //1

const app = express();
app.locals.projDir = \_\_dirname;

app.init = (config, logger) => {
  app.use( //2, use it as a middleware for all requests
    morgan('tiny', { //3 pass the middleware with log type and config
      stream: { //4 configure how to stream the request messages
        write: (message) => logger.info(message), //4 to our logger
      },
    })
  );
  app.get('/', (req, res) => {
    res.send(\`running in ${config.app.env} environment\`);
  });
};

module.exports = app;

You might notice that we send all messages as “info” level to Winston. That’s what we want because remember these are generic request information. Debugging, errors, etc we will handle with custom messages. These automatic ones should be of the “info” type.

There are various message formats, for example here are requests to “/” for our web server using three formats:

- **combined**: {“level”:”info”,”message”:”::1 – – \[08/Oct/2021:13:20:28 +0000\] \\”GET / HTTP/1.1\\” 200 397 \\”-\\” \\”Mozilla/5.0 (Macintosh; Intel Mac OS X 10\_15\_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/15.0 Safari/605.1.15\\”\\n”}
- **dev**: {“level”:”info”,”message”:”\\u001b\[0mGET / \\u001b\[32m200\\u001b\[0m 12.573 ms – 397\\u001b\[0m\\n”}
- **tiny**: {“level”:”info”,”message”:”GET / 200 397 – 1.981 ms\\n”}

Typically I’d keep Morgan request logs at “tiny” for normal usage and up the detail when doing investigation or for generating metrics with a 3rd party service like: solarwinds, datadog, or splunk.

The advantage here is that we once we set Morgan as middleware, every request will then be automatically logged, unlike the previous "Winston"-only example from earlier where we had to make explicit calls to the logger. You will still need to make explicit logging calls when necessary, however Morgan will automatically log standard ones.

You can find all changes since the last post [here](https://github.com/paultman/full-express-setup/compare/v1.1...v1.2)
