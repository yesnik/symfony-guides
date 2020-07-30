# Config

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

Look at `config/services.yaml`:

```yaml
services:
    # default configuration for services in *this* file
    _defaults:
        autowire: true      # Automatically injects dependencies in your services.
        autoconfigure: true # Automatically registers your services as commands, event subscribers, etc.
        public: false
        
    App\Service\MarkdownService:
        autowire: false
```

This is a special key `_defaults` sets default config values that 
should be applied to all services that are registered in this file.

We could set autowiring to `false` on just one service to override these defaults.

### Service auto-registration

Look at `config/services.yaml`:

```yaml
services:
    # makes classes in src/ available to be used as services
    # this creates a service per class whose id is the fully-qualified class name
    App\:
        resource: '../src/*'
        exclude: '../src/{Entity,Migrations,Tests}'
```

This makes all classes inside `src/` available as services in the container.

**Remember**: each class in the src/ directory is automatically registered as a service

But wait! Does that mean that all of our classes are instantiated on every single request? 
No: this line simply tells the container to *be aware* of these classes. 
But services are never instantiated until - and unless - *someone asks for them*. 
So, if we didn't ask for our `MarkdownService`, it would never be instantiated on that request.

**Important:** each service in the container is instantiated a maximum of once per request. 
If multiple parts of our code ask for the MarkdownHelper, 
it will be created just once, and the same instance will be passed each time. 
That's awesome for performance: we don't need multiple markdown helpers... 
even if we need to call parse() multiple times.

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

**kerner.secret**

If you ever need a cryptographic secret, Symfony has a parameter called `kernel.secret`.
We can use `%kernel.secret%` to get it in *yaml config* files.

### Envirinment variables

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

**Important**: Using `.env` isn't recommended for production is mostly because the logic to parse this file isn't optimized. 
So, you'll lose a small amount of performance - probably just a couple of milliseconds.

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
