# Symfony Messenger

[Messenger](https://symfony.com/doc/current/components/messenger.html) component helps applications send and receive messages to/from other applications or via message queues.

```bash
composer require symfony/messenger
```

Show messages and handlers:

```bash
php bin/console debug:messenger
```

Run worker to process images (add `-vv` option to see logs about consumed messages):

```bash
php bin/console messenger:consume
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
