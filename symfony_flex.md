# Symfony Flex

[Symfony Flex](https://github.com/symfony/flex) is a Composer plugin for Symfony. 
It automates the most common tasks of Symfony applications, 
like installing and removing bundles and other Composer dependencies. 

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
Click "Recipie" link, that will lead you to GitHub: https://github.com/symfony/recipes/tree/master/symfony/mailer

Unofficial packages host their recipies at https://github.com/symfony/recipes-contrib
