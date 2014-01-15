[![Build Status](https://travis-ci.org/mozilla/browserid-verifier.png?branch=master)](https://travis-ci.org/mozilla/browserid-verifier)

# A BrowserID verification server

This repository contains a flexible BrowserID verification server authored in
Node.JS.

# Getting Started

To run the verification server locally:

    $ git clone git://github.com/mozilla/browserid-verifier
    $ cd browserid-verifier
    $ npm install
    $ npm start

At this point, your verifier will be running and available to use locally over
HTTP.

# Configuration

There are several configuration variables which can change the behavior of the
server.  You can inspect available configuration variables in `lib/config.js`.
You can specify a set of `json` configuration files using the `CONFIG_FILES`
environment variable (separate each path with a comma (`,`)).  Finally, you can
inspect the current server configuration with:

    $ node ./lib/server.js -c

# Health Checks

The server exports an endpoint at `/status` that can be polled for server health.
When the server is healthy, a `200` HTTP response is returned with a body of `OK`.

# API

The server exports an HTTP endpoint at `/v2` that can be POSTed to verify BrowserID
assertions.  Arguments may be provided inside a JSON object.  The following are
required:

1. Requests must be an HTTP POST.
2. Content-Type must equal `application/json`
3. POST body must be valid JSON.

The following arguments are supported:

## **required** (string) `assertion`

A BrowserID assertion

## **required** (string) `audience`

The origin of the site to which the assertion is expected to be bound.

## **optional** (array of strings) `trustedIssuers`

An array of domain names that are *trusted* issuers.  Assertions
signed by any of the domains in this set will be honored regardless of
the presence of a subject or principal in a BrowserID assertion.

## Error Response

Example:

    $ curl -H 'Content-Type: application/json' \
      -d '{ "audience": "http://example.com", "assertion": "bogus" }' \
      https://verifier.mozcloud.org/v2
    {
      "status": "failure",
      "reason": "no certificates provided"
    }

Upon failure, the verifier returns a non-200 HTTP status code.  Additionally, the
response body contains a JSON formated response containing a `status` key with the
value of `failure`.  Additionally, more verbose developer readable information will
be available in a string value on the `reason` key.

## Success Response

Example:

    $ curl -H 'Content-Type: application/json' \
      -d '{ "audience": "http://123done.org" , "assertion": "eyJhbG...ZEe7A" }'
      https://verifier.mozcloud.org/v2
    {
      "audience": "http://123done.org",
      "expires": 1389791993675,
      "issuer": "mockmyid.com",
      "email": "lloyd@mockmyid.com",
      "status": "okay"
    }

Upon successful assertion verification, a 200 response will be sent with a JSON formatted body.
The body will always include `audience`, `issuer`, `status` (of "okay"), and `expires`. 

## Extra IdP claims

XXX: document me
