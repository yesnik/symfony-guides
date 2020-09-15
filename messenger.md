# Symfony Messenger

[Messenger](https://symfony.com/doc/current/components/messenger.html) component helps applications send and receive messages to/from other applications or via message queues.

```bash
composer require symfony/messenger
```

**Show messages and handlers**

```bash
php bin/console debug:messenger
```

**Run worker to process messages**

```bash
php bin/console messenger:consume -vv

# Consume messages from async transport
php bin/console messenger:consume -vv async
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

3. Tell Messenger to pass messages to *async* transport. Edit `config/packages/messenger.yaml`:

```yaml
framework:
    messenger:
        transports:
            async: '%env(MESSENGER_TRANSPORT_DSN)%'
        routing:
            # Route your messages to the transports
            'App\Message\AddPonkaToImage': async
```
