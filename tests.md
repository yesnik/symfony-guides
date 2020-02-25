# Tests

## Installation

Install `test-pack`, which comes with the *PHPUnit bridge*: a Symfony component that wraps around PHPUnit.

```bash
composer require test --dev
```

## Configuration

### File `phpunit.xml.dist` 

It holds sensible default config for PHPUnit itself.

One of the keys here is called `SYMFONY_PHPUNIT_VERSION`. 
You can leave this or change it to the latest version of PHPUnit, which for me is 8.5

```xml
<php>
    <!-- ... -->
    <server name="SYMFONY_PHPUNIT_REMOVE" value="" />
    <server name="SYMFONY_PHPUNIT_VERSION" value="8.5" />
</php>
```

When we use Symfony's PHPUnit bridge, we don't require PHPUnit directly. 
Instead, you tell it which version you want, and it downloads it in the background.

### File `.env.test`

It will only be loaded in the `test` environment. Define in this file credentials for test database:

```
DATABASE_URL=mysql://root:root@127.0.0.1:3306/api_platform_demo_test?serverVersion=5.7
```

## Console commands

**Run tests**

```bash
php bin/phpunit
```

**Create test database**

```bash
php bin/console doctrine:database:create --env=test
```
