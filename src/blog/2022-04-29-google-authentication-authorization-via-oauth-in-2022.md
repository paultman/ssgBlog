---
title: "Google Authentication & Authorization via Oauth in 2022"
date: "2022-04-29"
categories: 
  - "tech"
tags: 
  - "google"
  - "oauth2"
image: "image.png"
---

## Overview

I feel like I've had the misfortune of re-learning this topic over and over so I'll document the process this time. Besides, Google is discontinuing **Google Sign-In**:

> We are [discontinuing the Google Sign-In JavaScript Platform Library for web](https://developers.googleblog.com/2021/08/gsi-jsweb-deprecation.html). Beginning April 30th, 2022 new web applications must use the Google Identity Services library, existing web apps may continue using the Platform Library until the March 31, 2023 deprecation date. For authentication and user sign-in, use the new Google Identity Services SDKs for both [Web](https://developers.google.com/identity/gsi/web) and [Android](https://developers.google.com/identity/one-tap/android) instead.

So I have to do a new implementation anyway. The newer library is called "[Sign In With Google.](https://developers.google.com/identity/gsi/web)" I guess if you've got to rename it, why not just change the order of the words. Done.

Ok, why the change? First, they slightly improved security, lower friction between sign-in and sign-up, and have a more consistent experience across the web. The later likely has to do with Chrome browser sessions, and also the ability to notice that google users are already signed into Google, when they happen to been on other websites, and leverage that active session.

Another reason is to explicitly separate the to A's. **Authentication**, basically allowing a user to say who they are for sign-in/sign-up, and **Authorization**, which allows users access to Google Services like Drive, Calendar, etc via OAUTH2 tokens.

Previously my team didn't use the Google libraries directly, but relied on [Passport](http://www.passportjs.org) for authentication, and direct [REST](https://googleapis.github.io/HowToREST.html) calls for authorization and API access. Now though, we are moving to using Google's provided web javascript [client](https://accounts.google.com/gsi/client), and their Nodejs [library](https://github.com/googleapis/google-api-nodejs-client).

If you happen to be using apis.google.com/js/api.js, apis.google.com/js/client.js, or apis.google.com/js/platform.js, it's time to update to the newer, [Google Identity Services](https://developers.google.com/identity/gsi/web)

## Gotchas

In this post, I will not repeat what you can find on Googles A&A pages, however I will link to them, and I will also explain a few gotchas which were not clear and I only solved via online searches and tracing through the open source [Google's API](https://github.com/googleapis/google-api-nodejs-client) for Nodejs code.

Namely:

- How to connect the separate Authentication and Authorization user flows
- The OAuth refresh token behavior and how to explicitly request a new refresh token
- How API calls work, specifically related to handling expired Access Tokens and retries
- After an automatic retry due to expired token, how to save new token for future requests

My particular use case is to leverage Google for user sign in, and also to use a Google API to read and write Calendar events. With that said, this post can be applied to using any other Google services, besides Calendar, like Maps, Drive, Gmail, etc. At the end, you will understand how to show a login button which could also reflect an existing Google Session, and upon login, check for user tokens, show a consent window if necessary, and refresh an expired access token.

## Authentication

Ok, rather than writing your own user authentication management system, you've decided to use Google to handle that.

If you only need to use Google for Authentication, you should follow this guide:

[https://developers.google.com/identity/gsi/web/guides/overview](https://developers.google.com/identity/gsi/web/guides/overview)

As mentioned above, start by creating a project and get a [ClientID](https://developers.google.com/identity/gsi/web/guides/get-google-api-clientid). You'll also find the client button [code generator](https://developers.google.com/identity/gsi/web/tools/configurator) helpful. The advantage in leveraging the google client JS library, is that the button will be dynamic, rendered with existing Google session information else a generic button.

Follow the google instructions to create a projectHere is some example code, to keep things simple, and readable, i'm including JS in the html snippet:

<div
        class="g\_id\_signin"
        data-type="standard"
        data-shape="pill"
        data-theme="outline"
        data-text="signin\_with"
        data-size="medium"
        data-logo\_alignment="left"
      ></div>
      <button id="oauthBtn">Login with Google Account</button>

Basically, that is a synchronous script to get the Google Sign In client library, then I initialize the client using the client ID, scope(s), the ux\_mode, and the redirect\_url which should match the url you gave when registering your project.

Once Google does the authentication, you will want google to post information to your backend webserver. It will come in the form of a POST request and the body will contain a [JWT](https://developers.google.com/identity/gsi/web/reference/js-reference#CredentialResponse) which contains the logged in users information.

...
async function authCB(req, resp) {
  if (req.query.error) {
    return resp.send(req.query.error);
  }
  const { tokens } = await oauth2Client.getToken(req.query.code);
  const userInfo = (
    await oauth2Client.verifyIdToken({
      idToken: tokens.id\_token,
      audience: config.google.clientID,
    })
  ).payload;

  if (!UserIndex.email\[userInfo.email\]) {
    const newUser = {
      name: userInfo.name,
      email: userInfo.email,
      sub\_id: userInfo.sub,
      picture: userInfo.picture,
      r\_token: tokens.refresh\_token,
    };
...

Besides decoding the JWT, we check if we have the current email in our database, and if not, we create a new record.

## Authorization

We can do the Authorization in one of two ways. We can trigger it with a button click on a html page:

...
    <script src="https://accounts.google.com/gsi/client"></script>
    <script>
      let client;
      function initClient() {
        client = google.accounts.oauth2.initCodeClient({
          client\_id: 'xxxx-xxxxx.apps.googleusercontent.com',
          scope:
            'https://www.googleapis.com/auth/userinfo.profile \\
           https://www.googleapis.com/auth/userinfo.email'
          ux\_mode: 'redirect',
          redirect\_uri: 'http://localhost:5000/oauth2callback',
        });
      }
      // Request an access token
      function getAuthCode() {
        client.requestCode();
      }
      initClient();
      document.getElementById('oauthBtn').addEventListener('click', getAuthCode);
    </script>
...

The client code is similar to the the Authentication code above, however unlike using data attributes in the sign in button, you'll be setting parameters in javascript, for example you'll include a "scopes" array. A difference on the server-side callback request is that it is a GET operation, and unlike getting user profile information (like email, full name, etc), you'll need to specifically request 'https://www.googleapis.com/auth/userinfo.profile' and 'https://www.googleapis.com/auth/userinfo.email' scopes.

Or we can trigger the code in our initial sign-in server-side callback. After doing the sign-in, we check if we have a refresh token already for that user. If so we use that, if not, they are a new user, we authorize them by generating a URL and sending it to the client as a redirect (gotcha 2, solved):

...
  // check for refresh token
  if (!user.refresh\_token) {
    // get oauth2 tokens
    const url = oauth2Client.generateAuthUrl({
      access\_type: 'offline', // include refresh token in response
      scope: 'https://www.googleapis.com/auth/calendar.readonly https://www.googleapis.com/auth/calendar.events',
      login\_hint: user.sub, // use previously authenticated user, don't reprompt to select user
      prompt: user.refresh\_token ? 'none' : 'consent', // consent screen to force a new refresh token if necessary
    });
    return resp.redirect(url);
...

[https://cloud.google.com/nodejs/docs/reference/google-auth-library/latest/google-auth-library/generateauthurlopts](https://cloud.google.com/nodejs/docs/reference/google-auth-library/latest/google-auth-library/generateauthurlopts)

This is useful when linked with the client initiated sign-in Authentication button above. To link the two, notice the "login\_hint" parameter, this prevents account selection after having already done that previously. (gotta 1, solved)

With either the html/javascript or backend redirect, you'll need a backend endpoint to process the auth from Google. It will be a GET request and will contain a query parameter called code, which you will use to get your access token, refresh token (assuming you requested offline access), and user info via id\_token. To decode the id\_token which is in JWT format, you can call the verifyIdToken method of the oauth2Client object.

Here's a look how to handle the callback:

...
async function authorizeCB(req, resp) {
  if (req.query.error) {
    return resp.send(req.query.error);
  }
  const { tokens } = await oauth2Client.getToken(req.query.code);
  const user = indexBy.id\[req.session.id\];
  user.refresh\_token = tokens.refresh\_token;
  const result = await userCol
    .updateOne({ \_id: ObjectId(req.session.id) }, { $set: { refresh\_token: user.refresh\_token } })
    .catch((err) => {
      console.log(err);
    });
  indexBy.id\[req.session.id\].access\_token = tokens.access\_token;
  indexBy.id\[req.session.id\].expiry\_date = tokens.expiry\_date;

  return resp.redirect('/home.html');
....

## Handling Expired Access Tokens

If you are missing the access\_token, or it is expired (only have a 60min life), a request is automatically made by the google library in the preflight to get a valid access\_token (gotcha 3):

// oauth2client.js
....
    async refreshTokenNoCache(refreshToken) {
        if (!refreshToken) {
            throw new Error('No refresh token is set.');
        }
        const url = OAuth2Client.GOOGLE\_OAUTH2\_TOKEN\_URL\_;
        const data = {
            refresh\_token: refreshToken,
            client\_id: this.\_clientId,
            client\_secret: this.\_clientSecret,
            grant\_type: 'refresh\_token',
        };
        // request for new token
        const res = await this.transporter.request({
            method: 'POST',
            url,
            data: querystring.stringify(data),
            headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
        });
        const tokens = res.data;

Ok, so even without an access token, or with an expired one, the api calls will succeed. The last gotcha, was how to record the updated access token and expiration if they were automatically refreshed. Well after you make your api request, you will still have access to your oauth2Client, and you will find that the credentials property contains a new access\_token and expiry\_date (gotcha 4, solved), which was updated from the code above.

**Update**: You can attach an event listener to the oauth2Client object. Once a refresh of the access token happens, it will emit a 'tokens' event. To keep things in a single place, I'd recommend relying on this rather then doing multiple checks for the credentials property on api calls as I previously outlined above.

You can set the handler like this:

  oauth2Client.on('tokens', (tokens) => {
    const { email } = JSON.parse(atob(tokens.id\_token.split('.')\[1\])); //decode JWT and destructure email property
    indexBy.email\[email\].access\_token = tokens.access\_token;
    indexBy.email\[email\].expiry\_date = tokens.expiry\_date;
  });

Since it is in a separate request callback, I don't have access to the original user id, however the tokens returned include the id\_token, which is a JWT that I can decode and lookup which user the associated access\_token and expiration date applies to.

More information from Google:

[https://developers.google.com/identity/oauth2/web/guides/how-user-authz-works](https://developers.google.com/identity/oauth2/web/guides/how-user-authz-works)

[https://developers.google.com/identity/protocols/oauth2/openid-connect](https://developers.google.com/identity/protocols/oauth2/openid-connect)

[https://developers.google.com/identity/oauth2/web/guides/use-token-model](https://developers.google.com/identity/oauth2/web/guides/use-token-model)

While doing testing, you'll might need to sometimes remove granted access to your app, you can do that here:

[https://myaccount.google.com/permissions](https://myaccount.google.com/permissions)
