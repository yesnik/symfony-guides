# Logs

## Installation

Install [MonologBundle](https://github.com/symfony/monolog-bundle):

```bash
composer require symfony/monolog-bundle
```
Ensure that `MonologBundle` is included at `config/bundles.php`:

```php
return [
    // ...
    Symfony\Bundle\MonologBundle\MonologBundle::class => ['all' => true],
];
```

## Log in controller

```php
namespace App\Controller;

use Psr\Log\LoggerInterface;

class BlogController extends AbstractController
{
    #[Route('/api/blog', name: 'app_api_blog', methods: ['GET'], format: 'json')]
    public function index(LoggerInterface $logger): Response
    {
        $logger->info('Spam check result');
        $logger->error('An error occurred');
        $logger->critical('Redis is down', [
            // include extra "context" info in your logs
            'message_type' => 'report',
        ]);
    }
}
```

Logs are stored at `var/log/dev.log`

## Monolog config

And Monolog has a feature called *channels*, which are kind of like categories. 
Instead of having just one logger, you can have many loggers. 
Each has a unique name - called a channel - and each can do totally 
different things with their logs - like write them to different log files.

The main logger uses a channel called `app`. But other parts of Symfony are 
using other channels, like `request` or `event`.

How could we access one of these other Logger objects? 
I mean, when we use the `LoggerInterface` type-hint, it gives us the main logger. 
But what if we need a different Logger, like the *"event"* channel logger?

### Add a new chanel to monolog

We want that `markdown` channel exist in all environments, so edit file `/config/packages/monolog.yaml`:

```yaml
monolog:
    channels:
        - deprecation
        - markdown
```

It will make available the following service - `monolog.logger.markdown`:

```bash
php bin/console debug:container log
```
In the output we'll see `Psr\Log\LoggerInterface $markdownLogger`.

Let's tell Symfony to save logs from `markdown` channel to `markdown.log` file. Edit: `config/packages/monolog.yaml`:

```yaml
monolog:
    when@dev:
        handlers:
            # ...
    
            # It's our config
            my_markdown_logging:
                type: stream
                path: "%kernel.logs_dir%/markdown.log"
                # "debug" is lowest level - all messages will be logged
                # "error" - errors, critical messages will be logged
                level: debug
                channels: ["markdown"]
```

So how can we tell Symfony to not pass us the `main` logger, 
but instead to pass us the `monolog.logger.markdown` service? 

That's no problem: when autowiring doesn't do what we want, just correct it! 

#### Way 1. Variable name

```php
class MarkdownService
{
    public function __construct(
        private readonly LoggerInterface $markdownLogger
    ) {}
}
```

#### Way 2. Use attribute `WithMonologChannel`

```php
use Monolog\Attribute\WithMonologChannel;

#[WithMonologChannel('parser')]
class MarkdownService
{
    public function __construct(
        private readonly LoggerInterface $logger
    ) {}
}
```

#### Way 3. Config file

Edit `config/services.yaml`:

```yaml
services:
    # ...

    App\Service\MarkdownService:
        arguments:
            $logger: '@monolog.logger.markdown'
```

The argument we want to configure is called `$logger`. 
We tell the container what value to pass to that argument. Use the service id: `monolog.logger.markdown`. 
*Note*: Symbol `@` tells Symfony not to pass us that string, but to pass us the service with that id.
