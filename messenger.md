# Symfony Messenger

[Messenger](https://symfony.com/doc/current/components/messenger.html) component helps applications send and receive messages to/from other applications or via message queues.

```bash
composer require symfony/messenger
```

**Show messages and handlers**

```bash
php bin/console debug:messenger
```

**Run worker to consume messages**

```bash
symfony console messenger:consume -vv

# Consume messages from `async_priority_high` and then from `async` transport
php ./bin/console messenger:consume -vv async_priority_high async

# Use the `--time-limit` option to stop the worker when the given time limit (in seconds) is reached:
php ./bin/console messenger:consume async_priority_high async --time-limit=3600
```

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

Starting from Symfony 5.1, the Doctrine transport has moved to a separate package. Install it by running:

```bash
composer require symfony/doctrine-messenger
```

The Doctrine transport can be used to store messages in a database table. Edit `.env`:

```
MESSENGER_TRANSPORT_DSN=doctrine://default
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

**Important:** Your Message Handler class must implement `MessageHandlerInterface`. 

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
        # Uncomment this (and the failed transport below) to send failed messages to this transport for later handling.
        failure_transport: failed
        
        transports:
            async:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                retry_strategy:
                    delay: 500
            async_priority_high:
                dsn: '%env(MESSENGER_TRANSPORT_DSN)%'
                options:
                    queue_name: high
            failed: 'doctrine://default?queue_name=failed'
            # sync: 'sync://'
        routing:
            'App\Message\AddPonkaToImage': async_priority_high
            'App\Message\DeletePhotoFile': async
```

4. Run worker to consume messages from `async_priority_high` and then from `async` transport:

```bash
symfony console messenger:consume -vv async_priority_high async
```
