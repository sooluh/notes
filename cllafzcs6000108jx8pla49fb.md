---
title: "How I Handled Secure Session Management in Adonis 5"
datePublished: Mon Aug 14 2023 05:36:20 GMT+0000 (Coordinated Universal Time)
cuid: cllafzcs6000108jx8pla49fb
slug: session-management-adonis-5
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1698944534058/49880fc6-18b1-478f-9c2f-cb1092b2dbd3.png
tags: adonisjs, session

---

In some cases from client requests, I've been required to implement the "logout from all sessions/devices when changing passwords" feature on the websites I build. This is a rather unique feature, as in my ±7 years as a web developer, I've never really tried something like that.

I tried searching for references, and it turns out, in Laravel there's a feature for that! You can find it [here](https://laravel.com/docs/10.x/authentication#invalidating-sessions-on-other-devices). But honestly, I'm not too fond of Laravel, I don't know, maybe because I haven't given it a shot.

Since from the beginning, some client projects requesting this feature were built using Adonis, I gladly racked my brain to think about this not-too-complicated matter. I don't know what's behind the `logoutOtherDevices()` method in Laravel, so I just guessed to create it myself, and it worked.

## Creating Migration

The first thing I did was to create a table using migration, let's call this table `sessions`.

```typescript
import BaseSchema from "@ioc:Adonis/Lucid/Schema";

export default class extends BaseSchema {
  protected tableName = "sessions";

  public async up() {
    this.schema.createTable(this.tableName, (table) => {
      table.string("id").primary();
      table.string("ip_address", 45);
      table.text("user_agent");
      table
        .bigInteger("enhancer")
        .notNullable()
        .index()
        .references("users.uuid");
      table.timestamp("created_at").defaultTo(this.raw("CURRENT_TIMESTAMP"));
    });
  }

  public async down() {
    this.schema.dropTable(this.tableName);
  }
}
```

I'm not sure if this is the best practice because in Laravel itself, I didn't find this additional table, maybe everything is handled by the Auth class (related to sessions). Since I also don't quite understand the session management behind Adonis, it's fine.

> Alternatively, if storing sessions in the database feels too heavy, you can use Redis for this.

## Model

After creating the migration and running it using `node ace migration:run`, it's time to create the model. Since it will be applied to several controllers in the future, to make it easier to use, I need to create a Model.

```typescript
import { DateTime } from "luxon";
import AppBaseModel from "./AppBaseModel";
import { column } from "@ioc:Adonis/Lucid/Orm";

export default class Sessions extends AppBaseModel {
  @column({ isPrimary: true })
  public id: string;

  @column()
  public ipAddress: string;

  @column()
  public userAgent: string;

  @column()
  public enhancer: number;

  @column.dateTime({ autoCreate: true, autoUpdate: false })
  public createdAt: DateTime;
}
```

## Login Controller

Now it's time to implement it in the login controller. `await Sessions.create()` is executed when all validations have passed and the session login is established, below it, a redirection to the dashboard page can be done.

```typescript
import Sessions from "App/Models/Sessions";
import { HttpContextContract } from "@ioc:Adonis/Core/HttpContext";

export default class LoginController {
  // ...

  public async process(ctx: HttpContextContract) {
    // ...

    await Sessions.create({
      id: ctx.session.sessionId,
      ipAddress: ctx.request.ip(),
      userAgent: ctx.request.header("user-agent"),
      enhancer: user.id,
    });

    // ...
  }
}
```

So, when someone successfully logs into an account, all sessions will be recorded and stored in the sessions table. Since there's an `enhancer` column whose value is the ID of the logged-in user, it can be easily controlled based on the user's ID.

## When Changing Password

Lastly, as the case I presented from the beginning is "logout from all sessions/devices when changing passwords", now is the time!

```typescript
import Sessions from "App/Models/Sessions";
import Redis from "@ioc:Adonis/Addons/Redis";
import Database from "@ioc:Adonis/Lucid/Database";
import { HttpContextContract } from "@ioc:Adonis/Core/HttpContext";

export default class PasswordController {
  // ...

  public async process(ctx: HttpContextContract) {
    // ...

    const trx = await Database.transaction();

    user.password = payload.new; // new password from payload
    await user.useTransaction(trx).save();

    // get all sessions (based on the current user)
    const sessions = await Sessions.query().where("user", user.id);
    // delete all data where the user column contains the user's id
    await Sessions.query({ client: trx }).where("user", user.id).delete();

    try {
      await trx.commit();
    } catch (error) {
      await trx.rollback();

      // redirect with a failed flash here
    }

    // since I'm using Redis
    for (const session of sessions) {
      await Redis.del(session.id);
    }

    // ...
  }
}
```

If you're using cookies, use this code.

```typescript
for (const session of sessions) {
  ctx.response.clearCookie(session.id);
}
```

For those using files, I've tried it before but always failed, you can give it a shot if you find it. The concept is simply to delete the session file (in .txt format) at the file path where the session files are stored. You can see its `destroy` method [here](https://github.com/adonisjs/session/blob/6fcea7bb144de18028b1ea693bc7e837cd799fdf/src/Drivers/File.ts#L78C1-L78C1).

For those using memory? I don't care, it's hard to explain, you can explore it yourself [here](https://github.com/adonisjs/session/blob/6fcea7bb144de18028b1ea693bc7e837cd799fdf/src/Drivers/Memory.ts#L42).

## Conclusion

And that's it! I hope it can help, I've also opened a discussion and proposal regarding this on the official Adonis Discord. Just wait, hopefully, Adonis 6 will have this feature.

Thanks.