---
layout: post
title: "Use Kong and Auth0 to Authenticate API Clients"
description: "Kong allows you to easily manage, extend, and secure your APIs. Learn how you can integrate Auth0 with Kong for superior authentication and security."
date: 2016-11-24 08:30
category: Technical guide, Kong, Server
author:
  name: "Ado Kukic"
  url: "https://twitter.com/kukicado"
  mail: "ado@auth0.com"
  avatar: "https://s.gravatar.com/avatar/99c4080f412ccf46b9b564db7f482907?s=200"
design:
  image: https://cdn.auth0.com/blog/angular/logo.png
  bg_color: "#333333"
tags:
- kong
- auth0-kong
- jwt
related:
- use-nginx-plus-and-auth0-to-authenticate-api-clients
- critical-vulnerabilities-in-json-web-token-libraries
- announcing-auth0-ambassador-program
---

---

**TL;DR** Kong is an API gateway and management layer that allows you to easily manage, extend, and secure your APIs. Kong delivers a powerful layer of abstraction and with its modular architecture provides powerful plugins that can offload many common operations done by your application. Operations such as rate limiting, logging, and JWT verification can easily be delegated to Kong so you can focus on developing the unique features of your application.

---

[Kong](https://getkong.org/) is an open-source API gateway and management layer focused on delivering high performance and reliablity. Originally built by [Mashape](https://www.mashape.com/) to manage and extend over 15,000 microservices, the product is now open-source and available for everyone. Kong is widely used in organizations such as the [New York Times](http://www.nytimes.com/), [Healthcare.gov](https://healthcare.gov), and [Condè Nast](http://www.condenast.com/).

In this post, we are going to look at why you would want to use a service such as Kong for your API. We'll examine the benefits, use cases, and finally we'll build a simple API and manage it with Kong. We'll close the post with a discussion on security and how Auth0 can help. Let's get started.

## Why API Management?

Building modern applications and APIs is becoming a very complex task. New languages, frameworks, and methodologies are challenging established concepts and best practices. The fall of the monolith and rise of microservices and API's is driving this revolution.

The landscape is shifting towards microservices and APIs. These services expose platform agnostic endpoints that are consumed by web browsers, native mobile, and IoT devices. Developers have the difficult task of ensuring that their applications not only work as intended, but are highly available, performant, and scalable. 

This is where Kong comes in. Kong aims to address the availability, performance, and scalability of APIs so developers can spend their time building the unique features of their applications.

{% include tweet_quote.html quote_text="Kong is an API gateway and management layer that allows you to easily manage, extend, and secure your APIs." %}

## Kong - Open Source API Gateway

Kong aims to make the management of APIs simple and easy. Kong exposes a RESTful API of its own so that developers can easily provision, configure, and manage their APIs. Additionally, a modular based architecture allows Kong to enhance the capabilities of many APIs. For example, you can enable a rate-limiting plugin to prevent request overload or a logging plugin to get better stats on how your API is being utilized. Support for clustering and an easy to use CLI make Kong a powerful option for developers and network administrators alike.

![Kong](https://getkong.org/assets/images/docs/kong-architecture.jpg)

### Pre-requisites

Kong requires a datastore to hold your API and plugin configurations. You can use [PostgreSQL](https://www.postgresql.org/) or [Cassandra](http://cassandra.apache.org/) as your datastore. PostgreSQL is recommended for non-clustered instances, while Cassandra is preferred if you are building a clustered deployment.

For our tutorial today, we will use a PostgreSQL database. You can either install PostgreSQL locally on your machine, connect to an online instance, or what I'm going to do is create a PostgreSQL container using [Kitematic](https://kitematic.com/). Whatever method you choose, you'll need to create a database that you will use for your Kong configuration. I'll create a database named `kong`. The owner of the database will be the default user `postgres` with no password.

### Installing Kong

Kong is built on top of [NGINX](https://nginx.org/en/) and can run essentially anywhere. Just like NGINX, we can install it on our machine locally, host it in the cloud, or containerize it through Docker. For our tutorial, we'll just install it locally on our machine.

I will install Kong on my Mac using Homebrew. This is by far the easiest way to get up and running. If you don't already have Homebrew installed, you can get the instructions and additional info [here](http://brew.sh/). To install Kong, execute the following commands in your terminal of choice:

```sh 
brew tap mashape/kong
brew install kong
```

In a few seconds, Kong will be downloaded and installed on your machine. To ensure that Kong is installed, you can run the `kong version` command. At the time of this post, the stable version of Kong is 0.9.5.

![Verify Kong Installation](https://cdn.auth0.com/blog/auth0-kong/kong.png)

### Configuring Kong

Now that we have Kong installed, let's go ahead and configure our server. To do this, we'll need to create a `kong.conf` file. *Note: The file doesn't have to be named `kong`, it can be anything you want as long as it has the `.conf` extension.*

For our configuration, we'll just configure the database connection. You can see the defaults, as well as a get a sample `kong.conf` file from the [Kong GitHub repo](https://github.com/Mashape/kong). I'll go ahead and paste the [contents](https://github.com/Mashape/kong/blob/master/kong.conf.default) of the sample conf file in my `kong.conf` file. You'll notice that all of the settings are commented out. I'll go ahead and uncomment the database section starting around line 104.

```conf
# kong.conf

database = postgres             # Determines which of PostgreSQL or Cassandra
                                # this node will use as its datastore.
                                # Accepted values are `postgres` and
                                # `cassandra`.

pg_host = 192.168.99.100        # The PostgreSQL host to connect to.
pg_port = 32768                 # The port to connect to.
pg_user = postgres              # The username to authenticate if required.
pg_password =                   # The password to authenticate if required.
pg_database = kong              # The database name to connect to.
```

I'll leave the Cassandra settings commented out as we won't be needed them for this tutorial. We can now start up our Kong server and make sure it works. In your terminal navigate to the directory where your `kong.conf` file is stored and execute the following command:

```sh
kong start --config kong.conf --v
```

If all went well your terminal should look similar like the screenshot below. The last line should have read **Kong started**. You can confirm that the Kong server is running by navigating to the admin page at `localhost:8001`. If you get a response, then we are set. To stop the Kong server simply run the `kong stop` command in your terminal.

![Kong Started](https://cdn.auth0.com/blog/auth0-kong/kong-start.png)

The most likely error you could receive here is a PostgreSQL connection refused error. Ensure that you are providing the correct host, port, and other credentials. Also, be sure you are passing in the `--config` flag with your `kong.conf` file otherwise you're likely to run into some errors. In our command we also passed the `--v` flag, which stands for verbose and will give us a lot more info which will be useful for debugging if we run into any issues.

## Building the API

Now that we have Kong setup, let's go ahead and build the API that Kong will interface with. I'll just go ahead and build a simple API in Node.js that exposes a couple of different routes. You can see the implementation below:

```js
// server.js
// We'll define our dependencies
var express = require('express');
var jwt_decode = require('jwt-decode');

var app = express();

// Here we are adding a couple of different routes for our API
app.get('/', function(req, res){
	var decoded = jwt_decode(req.query.jwt);
	res.send('Welcome to the API.');
})

app.get('/api/leads', function(req, res){
	res.json({leads:[{id: 12345, name: 'ACME Corp'}, {id: 23456, name:'LeadGen'}, {id:45677, name: 'Organic Leads'}]})
})

app.get('/api/employees', function(req, res){
	res.json({employees:[{name: 'Ado Kukic', username:'@ado'}, {name: 'Diego Poza', username:'@tony'}, {name:'Prosper Otemuyiwa', username:'@unicodedeveloper'}, {name:'Sebastian Peyrott', username:'@speyrott'}, {name:'Kim Maida', username:'@kim.maida'}]})
})

// The /api/user route will decode a JWT and display its claims. More on that later.
app.get('/api/user', function(req, res){
	if(req.query.jwt){
		var user = jwt_decode(req.query.jwt)
		res.json(user);
	} else {
		res.send('No JWT Found');
	}
})

// Our API will listen on port 3000
app.listen(3000);
```

This will complete our simple API implementation. A real world application is likely to have other features such as middleware, logging, and error handling but for brevity we'll keep it short and to the point.

Let's start up this server by running `node server.js`. Navigate to `localhost:3000` to make sure that it works. You should see the message "Welcome to the API." displayed. Feel free to make sure all the other routes work as well before continuing. The `api/user` route should display the message "No JWT Found."

![No JWT Found](https://cdn.auth0.com/blog/auth0-kong/no-jwt-found.png)

## Managing Your API with Kong

Now that we have our API up and running, the next thing we are going to do is configure Kong to proxy requests from the Kong server to our API. We are going to do this using the RESTful API that Kong exposes. I will use cURL to send requests to the API, but you can use software like [Postman](https://www.getpostman.com/) if it's easier. Let's get started.

### Registering the API

The first thing we are going to do is register our API. To do that will will make a `POST` request to `localhost:8001/apis`. The command will look like this:

```sh
curl -i -X POST \
  --url http://localhost:8001/apis/ \
  --data 'name=node-api' \
  --data 'upstream_url=http://localhost:3000' \
  --data 'request_host=localhost'
```

Executing this command will register a new API called **node-api** with a proxy to `localhost:3000`. This means that now when we make a request to `localhost:8000`, which is the port that Kong listens on, our request will be proxied to `localhost:3000` and we will see the content "Welcome to the API" displayed. Test it to make sure that it works.

![Kong Proxy](https://cdn.auth0.com/blog/auth0-kong/proxy.png)

That's it. We're done. If all we wanted to do was proxy the requests, we would be done, but let's see what else Kong has to offer.

### Kong Plugins

Proxying requests is just one of the many things Kong can do for us. The team at Mashape has built a number of plugins that will make Kong infinitely more useful. Let's see some of these plugins in action.

#### Rate limiting

Every API should have some form of rate limiting to prevent abuse. But where in the stack should rate limiting occur? Some prefer it to be part of the request middleware, others think it's a network issue so should be handled by NGINX or services like Kong. I don't have the right answer as it really depends on how your application is built. What I can say though is that Kong makes rate limiting painless.

Let's enable rate limiting on our API. To do that we'll again make a request via cURL to our Kong server.

```sh
curl -X POST \
  http://localhost:8001/apis/node-api/plugins \
  --data "name=rate-limiting" \
  --data "config.hour=5"
```

Executing this command will enable the rate limiting plugin on our Node API and configure it so that we can make five requests to the API per hour. On the sixth and any subsequent request, we will get a HTTP 429 error saying that we have exceeded our quota.

![Kong Rate Limit Exceeded](https://cdn.auth0.com/blog/auth0-kong/rate-limit-exceeded.png)

Now our example was a bit extreme limiting users to five requests per hour. The Kong rate limiting plugin is highly configurable. We can set limits by minutes, seconds, or even months. There are many [additional configuration options](https://getkong.org/plugins/rate-limiting/) that can be configured to meet the majority of rate limiting use cases.

#### Bot Detection Plugin 

Let's enable another plugin. This time we'll enable the bot detection plugin. This plugin blocks requests from common crawlers such as Google, Facebook, or other website scrapers. We can enable this plugin by executing the following command:

```sh
curl -X POST \
  http://localhost:8001/apis/node-api/plugins \
  --data "name=bot-detection"
```

We can additionally pass parameters to blacklist or whitelist specific bots. It really is that simple. Kong has a number of prebuilt plugins that we can enable such as the two we've seen here. Other plugins include [IP Restriction](https://getkong.org/plugins/ip-restriction/), [CORS](https://getkong.org/plugins/cors/), [Request Transformers](https://getkong.org/plugins/request-transformer/), and [many others](https://getkong.org/plugins/).

#### Authentication Plugin

One final plugin I would like to introduce is the [JWT plugin](https://getkong.org/plugins/jwt/). As the name implies, this plugin verifies JSON Web Tokens and deals with authentication. Let's secure our API and make sure that only trusted clients can access it by enabling the JWT plugin. To enable the JWT plugin execute the following:

```sh
curl -X POST http://localhost:8001/apis/node-api/plugins \
  --data "name=jwt"
```

This will enable the JWT with the default configuration. The default configuration means that we'll be able to send JWT's either via an Authorization header or via a querystring named `jwt`. Additionally, our JWT will need to include an `iss` claim for the issuer.

Next, we'll need to create a consumer. This is essentially the client that will be consuming our API. To create a consumer execute the following:

```sh
curl -X POST http://localhost:8001/consumers \
    --data "username=<USERNAME>" \
    --data "custom_id=<CUSTOM_ID>"
```

For my `username` I will choose **Ado** and for the `custom_id` I will choose **ado**. Finally, you will need to create JWT credentials for this consumer. To do this, we'll again hit the Kong RESTful API.

```sh
curl -X POST http://kong:8001/consumers/ado/jwt \
  --data "key=adokey" \
  --data "secret=supersecret"
```

We now have everything we need to finalize our JWT implementation. If you were to navigate to `localhost:8000` now you would see a message saying **Unauthorized**. This is because we enabled the JWT plugin, which requires each request to the API to be accompanied by a valid token. We also created a consumer so now all we need to do is generate a valid JWT.

The easiest way to generate a JSON Web Token is through [jwt.io](https://jwt.io). Once you are on the site, clear the existing sample data and let's define two claims for our token. The first claim will be `iss` and for this claim we'll set our key which was `adokey`. Next, let's add a second claim called `name` and here just enter your name. Finally in the secret section, let's add the secret we chose, in my case it was `supersecret`.

![Generting JWT with jwt.io](https://cdn.auth0.com/blog/auth0-kong/jwt-io.png)

As you were making these changes you probably noticed that the string on the left hand side of the screen was changing with each keystroke. The [jwt.io](https://jwt.io) website created a new token each time we added a new claim or changed the secret. Let's copy the final token generated and go back to our app.

Navigate to `localhost:8000` and this time append a querystring `jwt` with the token you copied. The final request will look something like this:

```
localhost:8000?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhZG9rZXkiLCJuYW1lIjoiYWRvIn0.t86wrSIeBCBSRsJhsR6qRNPoiZo_dE5D_ZDMazL-o6M
```

Hit enter and now you should see the original message "Welcome to the API" displayed. What happened here was that when we made the request to the server, Kong extracted the JWT and verified that the token was valid using the key and secret we provided earlier and since the token was in fact valid, Kong allowed the request to go through.

Feel free to test out the other routes we have in our API and remember to append the `jwt` querystring with the token each time. The final route we wrote, `/api/user`, displays the token claims. Call this route to confirm that the claims you made on `jwt.io` are the same claims displayed on `localhost:8000/api/user`. Notice that when a request is proxied to our API, the querystring as well as some additional headers are passed along as well. Your API is now secure!

![JWT Verified](https://cdn.auth0.com/blog/auth-kong/consumer-ok.png)

## Aside: Integrating Auth0 with Kong

So far we've seen some of the awesome features Kong provides for managing and extending the capabilities of our API. We saw how you can use the JWT plugin to easily secure your API. To close out this tutorial, I want to show how you can combine Kong with Auth0 to make authenticating to your API even better.

The first thing we'll need to do is create a new consumer. We'll follow the same pattern as before:

```sh
curl -X POST http://localhost:8001/consumers \
    --data "username=<USERNAME>" \
    --data "custom_id=<CUSTOM_ID>"
```

For my `username` this time will be **Auth0** and the `custom_id` we'll set as **auth0**. For the next part, you will need to login to your Auth0 [management dashboard](https://manage.auth0.com). If you don't already have an account, [sign up](https://auth0.com/signup) for a free one now. From the Auth0 management dashboard, navigate to the **Clients** tab and either create a new app or select the default one. From here you will want to copy the **Domain** and **Client Secret**.

![Auth0 Management Dashboard](https://cdn.auth0.com/blog/auth0-kong/auth0-dashboard.png)

Next, we will set up the JWT credentials for the Auth0 client on our Kong server. For the `key` we will use the **Domain** and for the `secret` we will use the **Client Secret** we copied earlier.

```sh
curl -X POST http://kong:8001/consumers/auth0/jwt \
  --data "key=https://ado.auth0.com/" \
  --data "secret=aFGFzo9dRvT_jkw5ghisfnP_48NaBrovcJsq0M550eJnzujeb8Jr2tREoNePO1yQ"
```

So far so good. There is one final change we will need to make for Auth0 to work with the Kong JWT plugin. The Auth0 secret is base64 encoded, so we'll just need to configure our JWT plugin to treat the secret as a base64 encoded string. Luckily we can easily make this change using the Kong RESTful API.

```sh
curl -X PATCH http://localhost:8001/apis/api/plugins/b8368982-7e4f-400e-9535-915a7010f1e9 \
  --data "name=jwt" \
  --data "config.secret_is_base64=true"
```

We are now ready to test our implementation. Rather than using [jwt.io](https://jwt.io), let's use the [Auth0 Playground](https://auth0.github.io/playground/) to actually authenticate and retreive a token. The Auth0 Playground allows you to write and test code with the various versions of the [Auth0 Lock](https://auth0.com/lock) widget. We'll simply display Lock, have the user log in, and upon successfully authenticating we'll alert out the JWT. The implementation is below:

```js
// You can get the Client ID and Domain from the Auth0 Management Dashboard
var domain = 'adobot.auth0.com';
var clientID = 'E7v0bIDB2bM4ICfIgbWPbe6J6T54hsiT';

var lock = new Auth0Lock(clientID, domain);
lock.show({
  focusInput: false,
  popup: true
}, function (err, profile, access_token, state, token) {
  alert('JWT is: ' + access_token)
});
``` 

![Auth0 Playground Log In](https://cdn.auth0.com/blog/auth0-kong/auth0-playground.png)

Log in or sign up and you will see an alert displayed with a JWT. Copy this token, go back to your application, and navigate to `localhost:8000/api/user?jwt={JWT-YOU-COPIED}`. If all went well you will see something like the following screenshot.

![Kong with Auth0 Integration](https://cdn.auth0.com/blog/auth0-kong/auth0-jwt-ok.png)

Now you can have your users log in with Auth0 and use the token retrieved to make secure calls to your API. We didn't have to write any logic to verify the token in our Node.js application, we offloaded that responsibility to Kong. Auth0 provides an identity management solution including [social connections](https://auth0.com/docs/identityproviders), enterprise [signle-sign on](https://auth0.com/docs/sso/single-sign-on), [enhanced security features](https://auth0.com/docs/anomaly-detection), and [more](https://auth0.com/docs/overview).

## Conclusion

In this tutorial we showed how you can easily manage, extend, and secure your existing APIs with Kong. We were able to add functionality that would take weeks to manually implement with just a few calls via the Kong API. The API we built did not have to be concerned with rate limits or security, we let Kong and Auth0 that care of that, so we could focus on building and enhancing the core of our application.