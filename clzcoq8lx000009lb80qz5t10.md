---
title: "Deploying a Laravel App on Shared Hosting with CPanel"
datePublished: Fri Aug 02 2024 12:32:36 GMT+0000 (Coordinated Universal Time)
cuid: clzcoq8lx000009lb80qz5t10
slug: deploy-laravel-cpanel-shared-hosting
tags: laravel, cpanel

---

Shared hosting with CPanel has many limitations, especially if you're on a tight budget, and I find it challenging to deploy a Laravel app on it. Although some hosting providers offer packages with terminal access and even SSH, the cost compared to VPS/VM makes it more practical to just go with VPS/VM.

It's tough, but not impossible. A few weeks ago, I worked on a volunteer project using Laravel, and with limited funds, I could only afford the most budget-friendly shared hosting with CPanel. Since the project's scope was small and non-profit, we had to make do. Let's see what I did to deploy Laravel there.

You might have seen methods to deploy Laravel on shared hosting by separating the `public` directory to `public_html` and putting the rest outside `public_html`, then modifying the `index.php` script, which is complicated and messes up the structure, especially if you're using Git. Here, I'll show you how to keep using Git and update the source with a simple `git pull`.

## Clone Repository

Quite a long intro. Let's get straight to it. First, I cloned the repository. Wait, using what? There's no terminal! Using Cron Jobs.

Yes! Go to the Cron Jobs menu and create a new one with "Common Settings" set to "Once Per Minute". In another tab, open the File Manager and navigate to the `tmp` directory. In the "Command" field, enter the following command.

```bash
(git clone https://pat@github.com/username/repository.git /home/cpaneluser/website) >> /home/cpaneluser/tmp/deploy.log
```

Notice the "pat" part; you need to replace it with your Personal Access Token from GitHub, GitLab, or other services. Make sure it's read-only and specific to the related repository. Adjust "username" and "repository" as needed, and replace "cpaneluser" with your CPanel username.

After about a minute, switch to the file manager tab and click "Reload" in the `tmp` directory. If you see a new file named `deploy.log`, you can delete the Cron Job you just created to prevent the command from running multiple times.

## Delete `public_html` and Create a Symlink

Next, in the file manager, delete the default `public_html` directory. Then, create a symlink from `/home/cpaneluser/website/public` to `/home/cpaneluser/public_html`. Create a new Cron Job with the same Common Settings as before, and use the following command.

```bash
(ln -s /home/cpaneluser/website/public /home/cpaneluser/public_html) >> /home/cpaneluser/tmp/deloy.log
```

Wait for about a minute and then delete the Cron Job to prevent the command from running repeatedly.

## Install PHP Dependencies

Patience is key, as we have to wait for the Cron Job to run at the scheduled time. Let's continue.

Now, it's time to install PHP dependencies using Composer. Create a Cron Job with the same settings as before, and use the following command.

```bash
(cd /home/cpaneluser/website && /usr/local/bin/ea-php82 bin/composer.phar install --no-interaction --no-dev) >> /home/cpaneluser/tmp/deploy.log
```

You might notice that the command above changes the active directory to the git directory (`website`) and then runs `bin/composer.phar`? Right, I streamlined the process to minimize interaction with Cron Jobs.

You can download the `composer.phar` file from Composer's official [GitHub releases](https://github.com/composer/composer/releases) and upload it to your source code base in the `bin` directory. This simplifies the next steps, where we'll frequently interact with Composer.

Wait for about a minute and then delete the Cron Job to prevent it from running repeatedly.

## Migration (Fresh)

Before migrating, make sure the `.env` file is configured manually using the File Manager. You can create the database first in CPanel and then set up your `.env` file accordingly.

Then, create a new Cron Job with the same Common Settings and use the following command.

```bash
(cd /home/cpaneluser/website && /usr/local/bin/ea-php82 artisan migrate:fresh --seed) >> /home/cpaneluser/tmp/deploy.log
```

Warning!!! The command above uses `migrate:fresh`, so be careful!

Next, wait for about a minute and then delete the Cron Job to prevent it from running repeatedly.

## Create Storage Symlink

This is basic Laravel stuff. Create a Cron Job as before with the following command, and delete it after about a minute to prevent the command from running repeatedly.

```bash
(cd /home/cpaneluser/website && /usr/local/bin/ea-php82 artisan storage:link) >> /home/cpaneluser/tmp/deploy.log
```

## Optimize App

To optimize the application, create a Cron Job with the following command.

```bash
(cd /home/cpaneluser/website && /usr/local/bin/ea-php82 artisan down && /usr/local/bin/ea-php82 artisan optimize:clear && /usr/local/bin/ea-php82 artisan up) >> /home/cpaneluser/tmp/deploy.log
```

With the above command, I switch to the `website` directory, bring the Laravel application down, clear any existing optimizations, and then bring the application back up.

Delete the Cron Job after about a minute to prevent it from running repeatedly.

## Update Repository

Of course, if there's a new commit in the repository, we just need to `git pull` and the application will be updated to the latest version. Here's the command.

```bash
(cd /home/cpaneluser/website && /usr/local/bin/ea-php82 artisan down && git checkout . && git clean -f && git pull && /usr/local/bin/ea-php82 bin/composer.phar install --no-interaction --no-dev && /usr/local/bin/ea-php82 artisan optimize:clear && /usr/local/bin/ea-php82 artisan up && /usr/local/bin/ea-php82 artisan migrate --force) >> /home/cpaneluser/tmp/deploy.log
```

With the command above, you can understand that I bring the Laravel application down, then checkout to the latest branch and force clean any changes to avoid conflicts. You can adjust this if necessary and if it's considered unsuitable.

Then, I run the `git pull` command to get the latest code changes, followed by installing PHP dependencies without interaction and without development dependencies.

Next, in the command above, I clear the Laravel application optimizations, bring the Laravel application back up, and perform the latest migration, confirming "Yes" for migrations in production.

This command can be used to update the application to the latest code changes in the future. So, please note it down and create a Cron Job with the command above whenever you want to update the application.

## Conclusion

Very complicated? Absolutely! Is there a more practical way? Of course, by renting shared hosting with CPanel that has Terminal or remote SSH features. Or just go ahead and get a VPS/VM for more flexibility.

I hope this is helpful, thank you.