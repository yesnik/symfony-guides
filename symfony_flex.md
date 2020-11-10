# Symfony Flex

[Symfony Flex](https://github.com/symfony/flex) is a Composer plugin for Symfony. 
It hooks into the installation process. When it detects a package for which it has a recipe, it executes it.
It automates the most common tasks of Symfony applications, 
like installing and removing bundles and other Composer dependencies. 

The main entry point of a Symfony Recipe is a `manifest.json` file that describes the operations that need to be done to automatically register the package in a Symfony application. Look at [framework-bundle](https://github.com/symfony/recipes/tree/master/symfony/framework-bundle) recipe.

## Symfony Recipes Server

Link: https://flex.symfony.com/

This site allows us to run this command:

```
composer require mailer
```
On this site you can find recipe by alias. For example, `mailer` is alias for `symfony/swiftmailer-bundle`.

In your project `symfony.lock` will be created. This file is managed by Flex. 
It keeps track of which recipes have been installed. 
Whenever you install a package, Flex will execute the recipe for that package, if there is one. 
Recipes can add configuration files, create directories, or even modify files like `.gitignore` 
or `config/bundles.php` so that the library instantly works without any extra setup.

## Where do these Flex recipes live?

Visit https://flex.symfony.com/ and try to find "mailer". You'll see `symfony/mailer` package. 
Click "Recipe" link, that will lead you to GitHub: https://github.com/symfony/recipes/tree/master/symfony/mailer

Repository for recipes:

- Official: https://github.com/symfony/recipes
- Unofficial: https://github.com/symfony/recipes-contrib

## Console commands

- Show installed recipes in project:
```bash
composer recipes
```
- Show details about recipe:
```bash
composer recipes symfony/routing
```
