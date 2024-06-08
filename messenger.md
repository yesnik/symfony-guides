# Symfony Messenger

[Messenger](https://symfony.com/doc/current/components/messenger.html) component helps applications send and receive messages to/from other applications or via message queues.

When some logic should be executed asynchronously, send a message to a messenger bus. The bus stores the message in a queue and returns immediately to let the flow of operations resume as fast as possible.

A consumer runs continuously in the background to read new messages on the queue and execute the associated logic. The consumer can run on the same server as the web application or on a separate one.

```bash
composer require symfony/messenger
```

## Console commands

**Make message and message handler**

```bash
symfony console make:message
```
Enter name `CommentMessage`. Command will generate files:

- `src/Message/CommentMessage.php`
- `src/MessageHandler/CommentMessageHandler.php`

**Show buses, messages and handlers**

```bash
php bin/console debug:messenger
```

**Run worker to consume messages**

*Run worker in the background:*

```bash
symfony run -d --watch=config,src,templates,vendor symfony console messenger:consume async
```

The `--watch` option tells Symfony that the command must be restarted whenever there is a filesystem change in the config/, src/, templates/, or vendor/ directories.

*Run worker in the foreground:*

```bash
symfony console messenger:consume -vv

# Consume messages from `async_priority_high` and then from `async` transport
php ./bin/console messenger:consume -vv async_priority_high async
```

We can pass *option* to this command:

- `--time-limit=3600` - worker will stop when the given time limit in seconds is reached
- `--limit=10` - stop the worker if it exceeds a given limit of processed messages
- `--memory-limit=128M`- stop the worker if it exceeds a given memory usage limit. You can use shorthand byte values [K, M or G]

```bash
php ./bin/console messenger:consume async_priority_high async --time-limit=3600
```
Worker will be stoped *gracefully*, not in the middle of some operation.

**Stop workers gracefully**

```bash
php bin/console messenger:stop-workers
```
To send this signal, Symfony actually sets a flag in the cache system - and each worker checks this flag. 
If you have a multi-server setup, you'll need to make sure that your Symfony "app cache" is stored in something 
like Redis or Memcache instead of the filesystem so that everyone can read those keys.

**Show config for Messenger**

```bash
# Current config
php bin/console debug:config framework messenger

# Example config
php bin/console config:dump framework messenger
```

**Failed messages**

```bash
php bin/console messenger:failed:show

php bin/console messenger:failed:retry
```

**Show activated middlewares**

```bash
php bin/console debug:container --show-arguments messenger.bus.default.inner
```

## Doctrine Transport

The Doctrine transport can be used to store messages in a database table. 
Starting from Symfony 5.1, the Doctrine transport has moved to a separate package.

```bash
composer require symfony/doctrine-messenger
```

The Doctrine transport can be used to store messages in a database table. Edit `.env`:

```
MESSENGER_TRANSPORT_DSN=doctrine://default?auto_setup=0
```

The transport will automatically create a table `messenger_messages`.

## Example

1. Create file `src/Message/AddPonkaToImage.php`. 

*Note:* Try to pass only neccessary information to this class. For example, pass object's ID not the whole object.

```php
namespace App\Message;

class AddPonkaToImage
{
    private $imagePostId;

    public function __construct(int $imagePostId)
    {
        $this->imagePostId = $imagePostId;
    }

    public function getImagePostId(): int
    {
        return $this->imagePostId;
    }
}
```

2. Create file `src/MessageHandler/AddPonkaToImageHandler.php`. 

**Important:** Your Message Handler class must implement `MessageHandlerInterface` - a *marker* interface. It only helps Symfony auto-register and auto-configure the class as a Messenger handler. By convention, the logic of a handler lives in the `__invoke()` method. The `AddPonkaToImage` type hint on this method’s one argument tells Messenger which class this will handle.

```php
namespace App\MessageHandler;

use App\Message\AddPonkaToImage;
use App\Repository\ImagePostRepository;
use Symfony\Component\Messenger\Handler\MessageHandlerInterface;

class AddPonkaToImageHandler implements MessageHandlerInterface
{
    private $imagePostRepository;

    public function __construct(
        ImagePostRepository $imagePostRepository
    )
    {
        $this->imagePostRepository = $imagePostRepository;
    }

    public function __invoke(AddPonkaToImage $addPonkaToImage)
    {
        $imagePostId = $addPonkaToImage->getImagePostId();
        $imagePost = $this->imagePostRepository->find($imagePostId);

        // other actions
    }

}
```

3. Tell Messenger to pass messages to *async_priority_high* transport. Edit `config/packages/messenger.yaml`:

```yaml
framework:
    messenger:
        # after retrying, messages will be sent to the "failed" transport
        failure_transport: failed
        
        transports:
            async:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                retry_strategy:
                    max_retries: 2
                    # milliseconds delay
                    delay: 500
                    # causes the delay to be higher before each retry
                    # e.g. 1 second delay, 3 seconds, 9 seconds
                    multiplier: 3
                    
            async_priority_high:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
                    queue_name: high
                    
            failed: 'doctrine://default?queue_name=failed'

        routing:
            'App\Message\AddPonkaToImage': async_priority_high
            'App\Message\DeletePhotoFile': async
```

Messenger has a retry mechanism for when an exception occurs while handling a message. See `transports:failed` param.
If a problem occurs while handling a message, the consumer will retry 2 times before giving up. 
But instead of discarding the message, it will store it in a more permanent storage, the `failed` queue, which uses the Doctrine database.

4. Run worker to consume messages from `async_priority_high` and then from `async` transport:

```bash
php bin/console messenger:consume -vv async_priority_high async
```
