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

### Debug server

```bash
bin/console server:dump
```

All `dump()` calls will be showed in console. It's useful for debugging ajax-requests.

## Debug pack

Install [debug pack](https://github.com/symfony/debug-pack) that has Monolog:

```bash
composer require debug
```

### Logs

Log files can be found at `var/log/dev.log`
