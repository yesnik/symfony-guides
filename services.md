## Services

When you create a service class, the arguments to its constructor are **autowired**. 
That means that we can use any of the classes or interfaces from `debug:autowiring` as type-hints. 
When Symfony creates our `MarkdownService` it knows what to do:

File of service: `src/Service/MarkdownService.php`:

```php
namespace App\Service;

use Michelf\MarkdownInterface;
use Symfony\Component\Cache\Adapter\AdapterInterface;

class MarkdownService
{
    protected $cache;
    protected $markdown;

    public function __construct(AdapterInterface $cache, MarkdownInterface $markdown)
    {
        $this->cache = $cache;
        $this->markdown = $markdown;
    }

    public function parse(string $source): string
    {
        $item = $this->cache->getItem('markdown_'.md5($source));

        if (!$item->isHit()) {
            $item->set($this->markdown->transform($source));
            $this->cache->save($item);
        }

        return $item->get();
    }
}
```

In controller:

```php
namespace App\Controller;

use App\Service\MarkdownService;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class ArticleController extends AbstractController
{
    public function show($slug, MarkdownService $markdownService)
    {
        // ...
        $articleContent = $markdownService->parse($articleContent);
        // ...
    }
}
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

## Private vs Public Service

In Symfony 3, services were defined as *public*. 
This means that you could use a `$this->get()` shortcut method in your controller to fetch a service by its id. 
Or, if you had the container object itself - yep, that's totally possible - 
you could say `$container->get()` to do the same thing.

But in Symfony 4, most services are private. What does that mean? 
Very simply, when a service is private, you cannot use the `$this->get()` shortcut to fetch it.
