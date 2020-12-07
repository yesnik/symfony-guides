# Tests

## Installation

Install [test-pack](https://github.com/symfony/test-pack), which comes with the [PHPUnit bridge](https://symfony.com/doc/current/components/phpunit_bridge.html): a Symfony component that wraps around PHPUnit.

```bash
composer require test --dev
```

## Configuration

### File `phpunit.xml.dist` 

It holds sensible default config for PHPUnit itself.

One of the keys here is called `SYMFONY_PHPUNIT_VERSION`. 
You can leave this or change it to the latest version of PHPUnit, which for me is 9.0.

```xml
<php>
    <!-- ... -->
    <server name="SYMFONY_PHPUNIT_REMOVE" value="" />
    <server name="SYMFONY_PHPUNIT_VERSION" value="9.0" />
</php>
```

**Important**: 

1. When we use Symfony's PHPUnit bridge, *we don't require PHPUnit directly*. 
Instead, you tell it which version you want, and it downloads it in the background.
2. PHPUnit will be installed to `your_project_folder/bin/.phpunit/phpunit-9.0-0`

### File `.env.test`

It will only be loaded in the `test` environment. Define in this file credentials for test database:

```
DATABASE_URL=mysql://root:root@127.0.0.1:3306/api_platform_demo_test?serverVersion=5.7
```

## PHPStorm symfony/phpunit-bridge config

1. Open settings *Languages & Frameworks* > *PHP* > *Test frameworks*
2. Choose radio button *Path to phpunit.phar* and define *Path to phpunit.phar*: `your_project_folder/bin/phpunit`
3. Activate checkbox *Default configuration file:* `your_project_folder/phpunit.xml.dist`

## Console commands

**Run tests**

```bash
php bin/phpunit
```

**Create test database**

```bash
php bin/console doctrine:database:create --env=test
```

**Create database schema**

```bash
php bin/console doctrine:schema:create --env=test
```

## Reset database before test

Install a bundle that ensures that each test is run with the same unmodified database:

```
composer require --dev dama/doctrine-test-bundle
```

Enable it as a PHPUnit extension:

```xml
<!-- phpunit.xml.dist -->
<phpunit>
    <!-- ... -->

    <!-- Add this for PHPUnit 7.5 or higher -->
    <extensions>
        <extension class="DAMA\DoctrineTestBundle\PHPUnit\PHPUnitExtension"/>
    </extensions>
</phpunit>
```

## Useful methods

### `self::$container->get()`

From a PHPUnit test, it helps you to get any service from the container. It also gives access to non-public services.

```php
$comment = self::$container->get(CommentRepository::class)->findOneByEmail($email);
$comment->setState('published');
self::$container->get(EntityManagerInterface::class)->flush();
```
