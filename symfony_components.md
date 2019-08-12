# Useful Symfony components

[Symfony Components](https://symfony.com/doc/current/components/index.html) implement common features needed to develop websites. They are the foundation of the Symfony full-stack framework, but they can also be used standalone.

## symfony / asset

This [component](https://github.com/symfony/asset) manages URL generation and versioning 
of web assets such as CSS stylesheets, JavaScript files and image files.
Also it gives us `asset()` method in templates.

```
composer require asset
```

## symfony / serializer

The [serializer](https://github.com/symfony/serializer) component is meant to be used to turn objects into a specific format (XML, JSON, YAML, etc.) and the other way around. This command will install `serializer-pack` - serializer with additional components:

```
composer require serializer
```

## symfony / validator

The [validator](https://github.com/symfony/validator) component provides tools to validate values.

```
composer require validator
```
