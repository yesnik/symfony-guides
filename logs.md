# Logs

## Installation

Install [MonologBundle](https://github.com/symfony/monolog-bundle):

```bash
composer require log
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

class ConferenceController extends AbstractController
{
        /**
     * @Route("/conference/{slug}", name="conference")
     */
    public function show(Request $request, LoggerInterface $logger): Response {

        $logger->info('Spam check result');
        $logger->error('An error occurred');
        $logger->critical('Redis is down', [
            // include extra "context" info in your logs
            'message_type' => 'report',
        ]);
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

We want that `markdown` channel exist in all environments, so we create file `/config/packages/monolog.yaml`:

```
monolog:
    channels: ['markdown']
```

It will make available the following service - `monolog.logger.markdown`:

```
./bin/console debug:container log
```

Tell Symfony that you want to save logs from `markdown` channel to `markdown.log` file. 

Edit: `/config/packages/dev/monolog.yaml`:

```yaml
monolog:
    handlers:
        # ...

        # It's our config
        my_markdown_logging:
            type: stream
            path: "%kernel.logs_dir%/markdown.log"
            level: debug
            channels: ["markdown"]
            
        # ...
```

So how can we tell Symfony to not pass us the `main` logger, 
but instead to pass us the `monolog.logger.markdown` service? 
This is our first case where autowiring doesn't work.

That's no problem: when autowiring doesn't do what you want, just... correct it! 
Edit `config/services.yaml`:

```yaml
services:
    # ...

    App\Service\MarkdownService:
        arguments:
            $logger: '@monolog.logger.markdown'
```

The argument we want to configure is called `$logger`. We are telling the container what value to pass to that argument. Use the service id: `monolog.logger.markdown`. 
*Note*: Symbol `@` tells Symfony not to pass us that string, but to pass us the service with that id.
