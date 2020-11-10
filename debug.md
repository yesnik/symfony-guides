# Debug

## Prifiler Pack

Install [profiler pack](https://github.com/symfony/profiler-pack) for Symfony projects:

```bash
composer require profiler --dev
```

Shortcut for `dump($x);die` in view or controller:
```
dd($x);
```

## Debug pack

Install [debug pack](https://github.com/symfony/debug-pack) that has Monolog:

```bash
composer require debug
```

### Unpack a pack

```bash
composer unpack symfony/debug-pack
```

This command will extract packages from the pack.

### Logs

Logs will help us to investigate issues not only in development, but also in production.

```bash
composer require logger
```

Log files can be found at `var/log/dev.log`
