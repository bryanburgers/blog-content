I was **wrong**.

See, I love [JSON][json]. I love how simple it is. I love how it is easy to
represent, parse, and produce in nearly every language I use. I love how it has
become the de-facto interchange language of the web.

So naturally, I thought JSON was right for every situation. Because I love JSON,
I was so blinded that I thought its strengths made it a natural choice to use
for configuration files. But I was **wrong**.

Thankfully, there's a better option for configuration files. [YAML][yaml].

## Easier to use

I've had quite a bit of experience with using JSON files for configuration.
`package.json` works fine. Some of my own custom services use it. Everything
seemed to be, well, OK.

And then I was forced – yes **forced** – to use YAML for a Symfony project.

Through that process, I realized that it's just much easier to use YAML. Most
configuration is simple key-value pairs, and being able to just write `key:
value` is not only simple, but closely resembles most Linux configuration files.

Compare:

```yaml
database_host: my.database.example
database_port: 4218
database_username: web_app_user
database_password: IHLhbef1WYpwxg
database_database: awesome_web_app

port: 8080
```

to:

```json
{
  "database_host": "my.database.example",
  "database_port": 4218,
  "database_username": "web_app_user",
  "database_password": "IHLhbef1WYpwxg",
  "database_database": "awesome_web_app",

  "port": 8080
}
```

It's not huge, but YAML makes setting basic values simpler. Almost no need for
quotes (or for escaping quotes). No surrounding brackets to match. No commas to
remember. Just add a key, a colon, a value, and you're done.

## Fewer hangups

I can't tell you how many times I've done this. Take the JSON example from right
up there. And then pretend the `port` key is no longer required. So remove it
from the config file.

```json
{
  "database_host": "my.database.example",
  "database_port": 4218,
  "database_username": "web_app_user",
  "database_password": "IHLhbef1WYpwxg",
  "database_database": "awesome_web_app",
}
```

Restart the service that is being configured and... **error**.

If you've done a lot in JSON, you already know the problem. I removed the last
key, and now there's a dangling comma which makes the file *invalid json*. Ugh,
what a frustrating hangup to run into. So now I have to update the configuration
*again* and restart the service *again*.

I could just get better at life and not make that mistake again (*ha!* wishful
thinking). Or I could use YAML and not run into that hangup ever again.

And for the record, I have similar issues with matching curly braces.

Of course, YAML has some of its own hangups. But I seem to run into them less
frequently.

## Comments are allowed

The biggest win when choosing YAML for a configuration language over JSON is
comments.

You know, those things that let you describe what a configuration option means
or why it is set the way it is.

I mean, let's get this straight. I understand, and even sometimes agree, with
why Douglas Crockford forbid comments from JSON. That doesn't make JSON bad. In
fact, that's great for a data-interchange language. It just makes JSON a
less-good fit *for configuration files*.

```json
{
  "standalone": true,
  "secure_port": 8443
}
```

```yaml
# Whether the application serves requests directly, and
# must enable https and redirect. Set this to false if a
# TLS terminator is sitting in front of the application.
standalone: true
# Defaults to 443.
secure_port: 8443
```

Comments are also useful for temporarily removing a configuration value, but
keeping it around. If we take the same example from above and want to use the
default for `secure_port`, we need to completely remove it from the JSON (and
remove the trailing comma from `standalone`). Then what? If we want it back, we
have to remember it? Write it down in some other file? On a sticky note? With
YAML, we can just pop a comment in front of `secure_port`, allowing us to easily
restore it later.

```json
{
  "standalone": false
}
```

```yaml
standalone: false
# secure_port: 8443
```

---

I love JSON. But configuration files are meant for humans. And, as a human, I've
found that YAML configuration files are much easier to maintain than JSON.

[json]: http://www.json.org/
[yaml]: http://yaml.org/
