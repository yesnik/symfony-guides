## Services

A service is just a class that does work. When you create a service class, the arguments to its constructor are **autowired**. 
That means that we can use any of the classes or interfaces from `debug:autowiring` as type-hints. 

```bash
php bin/console debug:autowiring Markdown --all
```

Option `--all` shows your custom services in the Symfony container.

### Example: MarkdownHelper

Edit: `src/Service/MarkdownHelper.php`:

```php
namespace App\Service;

use Knp\Bundle\MarkdownBundle\MarkdownParserInterface;
use Symfony\Contracts\Cache\CacheInterface;

class MarkdownHelper
{
    private $markdownParser;
    private $cache;
    private $isDebug;

    public function __construct(
        MarkdownParserInterface $markdownParser,
        CacheInterface $cache,
        bool $isDebug
    )
    {
        $this->markdownParser = $markdownParser;
        $this->cache = $cache;
        $this->isDebug = $isDebug;
    }

    public function parse(string $source): string
    {
        if ($this->isDebug) {
            return $this->markdownParser->transformMarkdown($source);
        }

        return $this->cache->get('markdown_'.md5($source), function() use ($source) {
            return $this->markdownParser->transformMarkdown($source);
        });
    }
}
```

Edit `src/Controller/ArticleController.php`:

```php
namespace App\Controller;

use App\Service\MarkdownHelper;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class ArticleController extends AbstractController
{
    public function show($slug, MarkdownHelper $markdownHelper)
    {
        // ...
        $articleContentParsed = $markdownHelper->parse($articleContent);
        // ...
    }
}
```

Edit `config/services.yaml`:

```yaml
services:
    App\Service\MarkdownHelper:
        bind:
            $isDebug: '%kernel.debug%'
```

### Setter Injection

In `SlackClient`, we want to log a message. We already know how to do this: 
add a second constructor argument, type-hint it with `LoggerInterface` and, we're done!

But... there's another way to autowire your dependencies: *setter injection*. 
Setter injection is less common than passing things through the constructor, but sometimes it makes sense for optional dependencies - like a logger. 
What I mean is, if a logger was not passed to this class, we could still write our code so that it works. 
It's not required like the `Slack` client.

Anyways, here's how setter injection works: create a public function setLogger() with the normal `LoggerInterface $logger` argument:

```php
namespace App\Service;

use Nexy\Slack\Client;
use Psr\Log\LoggerInterface;

class SlackClient
{
    protected $slack;

    /**
     * @var LoggerInterface|null
     */
    protected $logger;

    public function __construct(Client $slack)
    {
        $this->slack = $slack;
    }

    public function sendMessage(string $from, string $message)
    {
        if ($this->logger) {
            $this->logger->info('Beaming a message to Slack!');
        }

        $message = $this->slack->createMessage()
            ->from($from)
            ->withIcon(':ghost:')
            ->setText($message);

        $this->slack->sendMessage($message);
    }

    /**
     * @required
     * @param LoggerInterface $logger
     */
    public function setLogger(LoggerInterface $logger)
    {
        $this->logger = $logger;
    }
}
```

**Note:** PHPDoc `@required` above `setLogger` function makes Symfony to call it after instantiating of `SlackClient`. 

## How Autowiring Works

Symfony puts all of services inside *service container* - an array of services, where each object has unique id.

```bash
# Show all services in the container
php bin/console debug:container

# Show services that can be accessed via autowiring
php bin/console debug:autowiring
```

When Symfony sees a type-hinted argument `Psr\Log\LoggerInterface $mdLogger`, it looks for a service in the container with this exact id. 
First it looks for a service whose id is the *type-hint + the argument name*. In this case it looks for a service with id `Psr\Log\LoggerInterface $mdLogger`.

We can create this binding in `config/services.yaml`:

```yaml
services:
    Psr\Log\LoggerInterface $mdLogger: '@monolog.logger.markdown'
```

Some services are just *aliases* to another service. 
If you ask for the `AdapterInterface` service, Symfony will give you the `cache.app` service.
Autowiring only works with class or interface type-hints.

### Named Autowiring

There are multiple services in the container that implement the same interface. 
It's possible to choose the one we want.

This command will show all implementations of `Psr\Log\LoggerInterface`:

```bash
php bin/console debug:autowiring log
```
Let's create 'markdown' logger chanel. Create `config/packages/monolog.yaml`:

```yaml
monolog:
    channels: ['markdown']
```

Use different argument's name to get access to this channel:

```php
class MarkdownHelper
{
    public function __construct(LoggerInterface $markdownLogger)
    {
        $this->logger = $markdownLogger;
    }
    
    public function parse(string $source): string
    {
        $this->logger->info('Hello!');
    }
}
```
This message will go to `markdown` channel.

### Bind service to argument

Edit `config/services.yaml`:

```yaml
services:
    _defaults:
        bind:
            Psr\Log\LoggerInterface $mdLogger: '@monolog.logger.markdown'
```

Prefix the service id with `@` to tell the Symfony that it's not a simple string.

The `bind` key will help to configure any argument that can't be autowired.

### Bind parameter to argument

In the controller:

```php
class ConferenceController extends AbstractController
{
    public function show(Request $request, string $photoDir): Response {}
}
```

Here `$photoDir` is a string and not a service. How can Symfony know what to inject here? 
The Symfony Container is able to store parameters in addition to services. 
Parameters are scalars that help configure services. These parameters can be injected into services explicitly, or they can be bound by name.

Edit `config/services.yaml`:

```yaml
services:
    _defaults:
        bind:
            $photoDir: "%kernel.project_dir%/public/uploads/photos"
            $akismetKey: "%env(AKISMET_KEY)%"
```

The bind setting allows Symfony to inject the value whenever a service has a `$photoDir` argument.

We certainly don’t want to hard-code the value of the Akismet key in this file, so we are using an environment variable instead (`AKISMET_KEY`).

It is then up to each developer to set a "real" environment variable or to store the value in a `.env.local` file:

```
AKISMET_KEY=abcdef
```

For production, a "real" environment variable should be defined.

### Bind arguments to service

Suppose we have a service at `src/Service/TextUniquenessCheck`:

```php
namespace App\Service;

class TextUniquenessCheck
{
    public function __construct(
        private readonly string $apiKey
    ) {}
    // ...
}
```

To bind API key to this service we need to edit `config\services.yaml`:

```yaml
services:
    App\Service\TextUniquenessCheck:
        $apiKey: '%env(CONTENT_WATCH_API_KEY)%'
```

We can define value at `.env`:

```
CONTENT_WATCH_API_KEY=1s111UVNi313S
```

### Bind different service for dev environment

We have these files at `src/Services/Esia/` folder: `EsiaInterface.php`, `Esia.php`, `EsiaDummy.php` (emulates responses).

We have `src/Controller/EsiaController.php`:

```php
class EsiaController extends AbstractController
{
    #[Route('/esia/auth/{id}', name: 'esia_auth')]
    public function auth(string $id, Esia\EsiaInterface $esia): RedirectResponse
    {
        $url = $esia->getAuthUrl($id);
        // ..
```

We want in `dev` environment use `EsiaDummy` implementation of `EsiaInterface`. To do this edit `config/services_dev.yaml`:

```yaml
services:
  app.services.esia:
    class: App\Services\Esia\EsiaDummy
```

After that edit `config/services.yaml`:

```yaml
services:
    # ...
    # the id is not a class, so it won't be used for autowiring
    app.services.esia:
        class: App\Services\Esia\Esia

    # the `@app.services.esia` service will be injected when
    # an `App\Services\Esia\EsiaInterface` type-hint is detected
    App\Services\Esia\EsiaInterface: '@app.services.esia'
```

## Config parameters

Show a list of the *parameters* in the container:

```bash
php bin/console debug:container --parameters
```

Add option `--env=prod` to see values for Production.

### Add param to a container

Edit `config/services.yaml`:

```yaml
parameters:
    cache_adapter: cache.adapter.apcu
```

Edit `config/services_dev.yaml` to define param value for *dev* environment:

```yaml
parameters:
    cache_adapter: cache.adapter.filesystem`
```

### Use param in a config

```yaml
framework:
    cache:
        app: '%cache_adapter%'
```

### Read param in a Controller

```php
$this->getParameter('cache_adapter');
```

## Private vs Public Service

In Symfony 3, services were defined as *public*. 
This means that you could use a `$this->get()` shortcut method in your controller to fetch a service by its id. 
Or, if you had the container object itself - yep, that's totally possible - 
you could say `$container->get()` to do the same thing.

But in Symfony 4, most services are private. What does that mean? 
Very simply, when a service is private, you cannot use the `$this->get()` shortcut to fetch it.
