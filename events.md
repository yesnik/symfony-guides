# Events

During the execution of a Symfony application, lots of event notifications are triggered. 
Your application can listen to these notifications and respond to them by executing any piece of code.

See [Built-in Symfony Events](https://symfony.com/doc/current/reference/events.html)

## Doctrine Events

Doctrine, the set of PHP libraries used by Symfony to work with databases, provides a lightweight event system to update entities during the application execution. These events, called *lifecycle events*, allow to perform tasks such as “update the createdAt property automatically right before persisting entities of this type”.

Doctrine triggers events *before/after* performing the most common entity operations (e.g. *prePersist/postPersist*, *preUpdate/postUpdate*) and also on other common tasks (e.g. loadClassMetadata, onClear).

### Ways to listen Doctrine events

1. **Lifecycle callbacks**. They are defined as methods on the entity classes and they are called when the events are triggered. 
They have better performance because they only apply to a single entity class, 
but you can’t reuse the logic for different entities and they don’t have access to Symfony services.
2. **Lifecycle listeners and subscribers**, they are classes with callback methods for one or more events and they are called for all entities.
They can reuse logic among different entities and can access Symfony services but their performance is worse because they are called for all entities.
3. **Entity listeners**, they are similar to lifecycle listeners, but they are called only for the entities of a certain class.
They have the same advantages of lifecycle listeners and they have better performance because they only apply to a single entity class.

### 1. Doctrine Lifecycle Callbacks

Lifecycle callbacks are defined as methods inside the entity you want to modify. 
For example, suppose you want to set a `createdAt` date column to the current date, but only when the entity is first persisted (i.e. inserted). To do so, define a callback for the `prePersist` Doctrine event:

```php
namespace App\Entity;

use App\Repository\CommentRepository;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass=CommentRepository::class)
 * @ORM\HasLifecycleCallbacks()
 */
class Comment
{
    /**
     * @ORM\PrePersist
     */
    public function setCreatedAtValue()
    {
        $this->createdAt = new \DateTime();
    }
}
```

### 2. Doctrine Lifecycle Listeners

Lifecycle listeners are defined as PHP classes that listen to a single Doctrine event on all the application entities. 
For example, suppose that you want to update some search index whenever a new entity is persisted in the database.

*(custom example here is needed)*

### 3. Doctrine Entity Listeners

Entity listeners are defined as PHP classes that listen to a single Doctrine event on a single entity class.
For example, suppose that we want to call `computeSlug()` method whenever the conference is updated.

1. Define a listener for the `prePersist` and `preUpdate` Doctrine events. Create file `src/EntityListener/ConferenceEntityListener.php`:

```php
namespace App\EntityListener;

use App\Entity\Conference;
use Doctrine\ORM\Event\LifecycleEventArgs;
use Symfony\Component\String\Slugger\SluggerInterface;

class ConferenceEntityListener
{
    private $slugger;

    public function __construct(SluggerInterface $slugger)
    {
        $this->slugger = $slugger;
    }

    public function prePersist(Conference $conference, LifecycleEventArgs $event)
    {
        $conference->computeSlug($this->slugger);
    }

    public function preUpdate(Conference $conference, LifecycleEventArgs $event)
    {
        $conference->computeSlug($this->slugger);
    }
}
```

Note that the slug is updated when a new conference is created (`prePersist()`) and whenever it is updated (`preUpdate()`).

2. The next step is to enable the Doctrine listener in the Symfony application by creating a new service for it and tagging it with the `doctrine.orm.entity_listener` tag. Edit `config/services.yaml`:

```yaml
services:
    # add more service definitions when explicit configuration is needed
    # please note that last definitions always *replace* previous ones
    App\EntityListener\ConferenceEntityListener:
        tags:
            - { name: 'doctrine.orm.entity_listener', event: 'prePersist', entity: 'App\Entity\Conference'}
            - { name: 'doctrine.orm.entity_listener', event: 'preUpdate', entity: 'App\Entity\Conference'}
```

**Note:** Don’t confuse Doctrine event listeners and Symfony ones. Even if they look very similar, they are not using the same infrastructure under the hood.

## Creating an Event Subscriber

Another way to listen to events is via an *event subscriber*, 
which is a class that defines one or more methods that listen to one or various events. 
The main difference with the *event listeners* is that *subscribers always know which events they are listening to*.

```bash
symfony console make:subscriber TwigEventSubscriber
```

The command asks you about which event you want to listen to. Events:

- `Symfony\Component\HttpKernel\Event\ControllerEvent` - is dispatched just before the controller is called.

This command will create file `src/EventSubscriber/TwigEventSubscriber.php`.

### Show event's listeners

Run this command to find out which listeners are registered for `kernel.controller` event and their priorities:

```bash
symfony console debug:event-dispatcher kernel.controller
```
