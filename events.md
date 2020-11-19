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
