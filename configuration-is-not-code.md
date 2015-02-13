Configuration does not belong in the database. We've known this for years. For ExpressionEngine, [Master Config][masterconfig] helps us get this right. Pat Pohler has a great [introduction to this][pp].

But configuration also does not belong in git. Just like there should be a separation between content and code, there should also be a separation between code and configuration. Anything that can change between environments is configuration, and should not be version controlled.

> An app’s config is everything that is likely to vary between deploys (staging, production, developer environments, etc).
>
> Config varies substantially across deploys, code does not.
>
> —[The Twelve Factor App][12factor]

There are many good reasons why configuration should not be tracked.

1.    **Secrets stay secret**. If you don't check critical credentials – database logins, API keys, etc. – into git, they don't accidentally get leaked into the wild.

    Where I work, we recently received a repo from a local competitor because the client switched to us. That repo contained the root username and password for their database because they checked it in. Now I have it.

    > A litmus test for whether an app has all config correctly factored out of the code is whether the codebase could be made open source at any moment, without compromising any credentials.
    > —[The Twelve Factor App][12factor]

    Sometimes, we as professionals have a lot to do, so we forget to sanitize a repo before sharing it. That's reality. But if critical credentials are never in the repo, then we never leak them.

1.    **Local environments differ**. Your team members' local environments all differ. He uses a different database naming scheme. She actually set a password for her local SQL install.

    That's OK. If config isn't checked into git, you don't have to fight over whose is committed or leave your changes unstaged and try to remember not to commit them. Or worse, have a separate config file checked in for everybody who has ever worked on the site.

    To be fair, [Master Config][masterconfig] gets this part right:

    > What makes this convenient is that we ignore our `config.local.php` file from Git. That way each local developer has their own config overrides that won’t affect the main repository.

1.    **Portability**. There are tons of reasons you might want to change configuration. You moved a site to a new server. You temporarily put the site on a new server to evaluate a new hosting provider. You hired an intern and want him to change one small bit of CSS in a local environment. You brought in a consultant to attack a particularly tricky problem.

    You don't need a commit to track all of these situations, because it isn't important in the history. You should be able to download the site code and get it running without first having to change a tracked file.

1.    **External scripts are easier**. When you have a central, tracked config file, you need a way to determine which environment to use. [Master Config][masterconfig] uses `HTTP_HOST` to make this choice. But cron jobs and external scripts don't have an `HTTP_HOST`.

    When your configuration is always stored the same way and in the exact same place, but differs per environment, external scripts can rely on this without needing complicated logic to try to guess the environment.

1.    **Bisectable git history**. I love `git bisect`. It helps me find issues fast.

    The other day, I was trying to debug an issue with one of our sites. So of course I first tried `git bisect` to determine when the issue started occurring. But my configuration was checked into git _and_ it changed recently. So every time I went back in history, my configuration was wrong.

    If my configuration file was ignored by git, this would not have been a problem. Git wouldn't try to change my config file for every step of the `git bisect`.

## The Path Forward

So what should we do instead? Well, we should still use [Master Config][masterconfig]. It's awesome. Seriously. I don't know what we'd do without it.

What we do at [Click Rain][clickrain] is drop all the `config.whatever.php` files that deal with a specific environment, and we're left with `config.master.php` and `config.env.php`. Then we `.gitignore` the `config.env.php` so it's only local to the environment and define only configuration that changes between environments.

```php
<?php

if ( ! defined('ENV'))
{
    define('ENV', 'local');
    define('ENV_FULL', 'Local');
    define('ENV_DEBUG', 'true');

    define('DB_HOST', 'localhost');
    define('DB_NAME', 'awesomesite');
    define('DB_USERNAME', 'awesomeuser');
    define('DB_USERNAME', 'qfJZ6yXj17hix249cBiu3g');
}
```

The advantage of only using `define` in the config file is that we can easily load it from external scripts and cron jobs and use the values there as well.

We also want to make it easy to get started when you pull the repo for the first time. We check in a `config.env.php.sample` so that you can quickly copy it to `config.env.php` and enter your configuration details.

---

Configuration isn't code. It's separate and shouldn't be treated like code. We need to start treating configuration like what it is: something that's fluid and changes between environments. We need to start making our lives easier and stop tracking configuration into our code revision repositories.

[clickrain]: http://clickrain.com
[pp]: http://www.patpohler.com/expressionengine-multiple-environments/
[masterconfig]: https://github.com/focuslabllc/ee-master-config
[12factor]: http://12factor.net/config
