# Events

During the execution of a Symfony application, lots of event notifications are triggered. 
Your application can listen to these notifications and respond to them by executing any piece of code.

See [Built-in Symfony Events](https://symfony.com/doc/current/reference/events.html)

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
