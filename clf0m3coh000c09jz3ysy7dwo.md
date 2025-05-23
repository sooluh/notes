---
title: "Minifying HTML Output in AdonisJS"
datePublished: Thu Mar 09 2023 04:31:29 GMT+0000 (Coordinated Universal Time)
cuid: clf0m3coh000c09jz3ysy7dwo
slug: minifying-html-output-in-adonisjs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1747102737881/0e621fd4-3b07-4657-8cc4-ab136060a6f0.jpeg
tags: adonisjs, middleware

---

Let's start this AdonisJS series with a fairly light topic. **How to minify HTML output in AdonisJS**?

The convenience of AdonisJS, in my opinion, is partly due to AdonisJS being in the Node.js environment, and there are lots of packages that make it easier for us to work.

For this purpose, there is already an [html-minifier](https://www.npmjs.com/package/html-minifier) package, make sure you are in your AdonisJS project directory, then type the following command to install the package.

```bash
yarn add html-minifier
```

For development purposes, maybe you need `@types` for more convenience, so there are suggestions and autocomplete when using this package. Run the following command, making sure the -D flag is present, indicating that this is a dev dependency.

```bash
yarn add -D @types/html-minifier
```

In the next step, we're gonna need to create a middleware, we'll use this so that we don't have to repeat the same code in every controller. Ah, I'm sure you already understand the true function of middleware.

```bash
node ace make:middleware Minifier
```

Then we need to add it to `kernel.ts`, this is to register our middleware as global, so we don't need to call it individually or manually.

```typescript
// start/kernel.ts

Server.middleware.register([
  () => import('@ioc:Adonis/Core/BodyParser'),
  () => import('App/Middleware/Minifier'),
])
```

And the last step, we mix the code that will be pasted into the Minifier middleware. We've already commented on the code, so don't forget to try to understand what each line of code means and does.

```typescript
// app/Middleware/Minifier.ts

import { minify } from 'html-minifier'
import { types } from '@ioc:Adonis/Core/Helpers'
import Application from '@ioc:Adonis/Core/Application'
import { HttpContextContract } from '@ioc:Adonis/Core/HttpContext'

export default class MinifierMiddleware {
  public async handle({ response, request }: HttpContextContract, next: () => Promise<void>) {
    // here for altering, authorizing and preparing the request

    await next()

    // here for altering response

    // retrieve method used on the request
    const method = request.method()
    // retrieve desired content on request
    const accepts = request.accepts([]) ?? ([] as string[])

    // if method is not GET or except HTML or not in production, then exit
    if (method !== 'GET' || !accepts.includes('text/html') || !Application.inProduction) {
        return
    }

    // retrieve content in body to respond
    const body = response.getBody()

    // check data type, maybe this is an object or an array
    if (types.isObject(body) || types.isArray(object)) {
        return
    }

    // if everything is safe, then we can minify html output we get
    const minified = minify(body, {
        minifyCSS: true,
        minifyJS: true,
        removeComments: true,
        preserveLineBreaks: false,
        collapseInlineTagWhitespace: false,
        collapseWhitespace: true,
    })

    // set response to be minified html
    response.send(result)
  }
}
```

And it's done, thanks.