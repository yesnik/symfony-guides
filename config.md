# Config

To create a container Symfony needs to get all services that should be in the container - service's id, class name and the arguments. 
Symfony gets info about services from 2 places:

1. From bundles. Each bundle has a list of its services.
2. From `src/` directory. To learn about our services Symfony reads `services.yaml`. Everything in `src/` is automatically available as a service in the container.

## Config files

Symfony loads everything from `config/packages` and then loads the files in the environment subdirectory. 
It allows us to override the original values.

## Bundle config

Each file in `config/packages/` configures a some bundle. Any configuration under `knp_markdown` is passed to the `KnpMarkdownBundle`. Any config under `framework` configures `FrameworkBundle`, which is Symfony's one, "core" bundle.

### Show bundle config options

```bash
# Show config example for Main bundle of Symfony (includes cache, etc.)
php bin/console config:dump-reference FrameworkBundle

# Show current config values
php bin/console debug:config FrameworkBundle
```

Also we can use alias:

```
php bin/console config:dump-reference knp_markdown
```

Copy this bundle's config to file `config/packages/knp_makrdown.yaml`:

```yaml
knp_markdown:
  parser:
    service: markdown.parser.light
```

**Important:** We may need to clear cache to apply changes in settings: `php bin/console cache:clear`

## Services config

When Symfony loads, it needs to figure out all of the services that should be in the container. 

### Service auto-registration

Look at `config/services.yaml`:

```yaml
services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true      # Automatically injects dependencies in your services.
        autoconfigure: true # Automatically registers your services as commands, event subscribers, etc.
        bind:
            bool $isDebug: '%kernel.debug%'

    # makes classes in src/ available to be used as services
    # this creates a service per class whose id is the fully-qualified class name
    App\:
        resource: '../src/*'
        exclude: '../src/{DependencyInjection,Entity,Migrations,Tests,Kernel.php}'
```

This makes all classes inside `src/` available as services in the container.

Key `_defaults` sets default config values that 
should be applied to *all services* that are registered in this file.
We could set autowiring to `false` on just one service to override these defaults.

When we add a `bind` to `_defaults`, we can use `$isDebug` argument in *any* of our services.

Does that mean that all of our classes are instantiated on every single request? 
No: this line simply tells the container to *be aware* of these classes. 
But services are never instantiated until - and unless - *someone asks for them*. 
So, if we didn't ask for our `MarkdownHelper`, it would never be instantiated on that request.

**Important:** 

- each service in the container is instantiated a maximum of once per request. 
If multiple parts of our code ask for the `MarkdownHelper`, 
it will be created just once, and the same instance will be passed each time. 
- every service in the container needs to have a *unique id*. 
When we auto-register services in `src` directory, the `id` matches the class name. 
- If we try to autowire `App\Service\MarkdownHelper` into our controller or another service, 
autowiring looks in the container for a service whose `id` exactly matches the type-hint: `App\Service\MarkdownHelper`.

### Bind service to variable name

Edit `config/services.yaml`:

```yaml
services:
    # default configuration for services in *this* file
    _defaults:
        bind:
            $markdownLogger: '@monolog.logger.markdown'
```
This config tells: if you find any argument named `$markdownLogger`, pass this service to it.
And because we added it to _defaults, it applies to all our services. 
Instead of configuring our services one-by-one, we're creating project-wide conventions. 
Next time you need this logger? Yep, just name it `$markdownLogger` and keep coding.

### Bind class name to service

Edit `config/services.yaml`:

```yaml
services:
    Nexy\Slack\Client: '@nexy_slack.client'
```

It allows us to autowire class `Nexy\Slack\Client`.

By putting this config directly under services, we're creating a *new service* in the container 
with the id `Nexy\Slack\Client`. But this is not a real service, it's just an "alias" - a "shortcut" - 
to fetch the existing `nexy_slack.client` service.

**Important**: when an argument to a service hasn't been configured under bind or arguments, 
how does the autowiring figure out which service to pass? The answer is super simple: 
it just looks for a service whose id exactly matches the type-hint. 
Yep, now that there is a service whose id is `Nexy\Slack\Client`, 
we can use that class as a type-hint. 
That's also why our classes - like `MarkdownHelper` can be autowired: 
each class in `src/` is auto-registered as a service and given an id that matches the class name.

### Parameters

Edit common config file `config/services.yaml`:

```yaml
parameters:
    cache_adapter: cache.adapter.apcu
```

For *development environment* we create `config/services_dev.yaml`:

```yaml
parameters:
    cache_adapter: cache.adapter.filesystem
```

After that we can use this *cache_adapter* param in `config/packages/framework.yaml`:

```yaml
framework:
    cache:
        app: '%cache_adapter%'
```

**Remember**: anything surrounded by percent signs is a parameter (eg. `cache_adapter%`)

**kernel.secret**

If you ever need a cryptographic secret, Symfony has a parameter called `kernel.secret`.
We can use `%kernel.secret%` to get it in *yaml config* files.

### Envirinment variables

After Symfony loads `.env`, it looks for file `.env.local`.
Variables in this file will override the values in `.env`.

**Set variable**

Edit `.env`:

```yaml
### CUSTOM VARS
SLACK_WEBHOOK_ENDPOINT=https://hooks.slack.com/services/26/B91D3PPH/BX20IHE2
### END CUSTOM VARS
```

To save the name of this variable to version control edit `.env.dist`:

```yaml
### CUSTOM VARS
SLACK_WEBHOOK_ENDPOINT=https://hooks.slack.com/services/...
### END CUSTOM VARS
```

**Read variable**

Edit `config/packages/nexy_slack.yaml`:

```yaml
nexy_slack:
    endpoint: '%env(SLACK_WEBHOOK_ENDPOINT)%'
```

**Note**: Use this syntax to get environment var in yaml config: `%env(APP_SECRET)%` 

**Cast variables**

You can cast values by prefixing the name with, for example, `string:`, `bool:`, `float:`:

```
nexy_slack:
    endpoint: '%env(string:SLACK_WEBHOOK_ENDPOINT)%'
    is_verbose: '%env(bool:SLACK_IS_VERBOSE)%'
```

**Chain prefixes**

We can open a file, and then decode its JSON:
 
```
app.secrets: '%env(json:file:SECRETS_FILE)%'
```

### Secret vault

Symfony has *secrets vault* - a collection of encrypted values. It allows us to commit sensitive values into Git.

Symfony has two vaults:

- for the dev environment. It contains non-sensitive default values
- for the prod environment. It contains the real values.

**Create the vault**

```bash
php bin/console secrets:set SENTRY_DSN
```
This command will:

- create the file `config/secrets/dev/dev.SENTRY_DSN.25b27f.php`.
- initialize the `dev` vault:
  * `dev.encrypt.public.php` file contains the key that is used to encrypt secrets.
  * `dev.decrypt.private.php` file contains the key to decrypt the secrets and use them in our app. It's OK to commit the decrypt key for the `dev` vault, but not for the `prod` vault.

```bash
php bin/console secrets:set SENTRY_DSN --env=prod
```
This command will create the file for this secret and initialize `prod` vault.

*Important:* Don't commit `prod.decrypt.private.php` to Git.

Show secret values for `prod` vault:

```bash
php bin/console secrets:list --reveal --env=prod
```

**.env variables and Secrets**

If we use `%env(SENTRY_DSN)%` syntax, Symfony first looks to see if an environment variable called `SENTRY_DSN` exists. 
If there is not, it then looks for a secret in the vault called `SENTRY_DSN`. 
You should set a value as an environment variable or a secret, but not both.

**Why do we need Secret Vault?**

All secrets are stored in the repository and the only environment variable you need to manage for production is the *decryption key*. 
That makes it possible for *anyone in the team to add production secrets* even if they don’t have access to production servers.
