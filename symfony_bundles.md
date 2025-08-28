# Symfony Bundles

[Bundle](https://symfony.com/doc/current/bundles.html) is a collection of code and other files written for use in a Symfony application. 
They are dependent from Symfony components.

## AliceBundle

[AliceBundle](https://github.com/hautelook/AliceBundle) is a Symfony bundle to manage fixtures with Alice and Faker.

```bash
composer require --dev hautelook/alice-bundle
```

## EasyAdminBundle

[EasyAdminBundle](https://github.com/EasyCorp/EasyAdminBundle) is new and simple admin generator for Symfony apps.

```bash
composer require admin
```

## FOSOAuthServerBundle

[FOSOAuthServerBundle](https://github.com/FriendsOfSymfony/FOSOAuthServerBundle) is a server side OAuth2 Bundle for Symfony/

```bash
composer require friendsofsymfony/oauth-server-bundle
```

## FOSRestBundle

[FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle) provides various tools to rapidly develop RESTful API's with Symfony:

```bash
composer require friendsofsymfony/rest-bundle
```

## FOSUserBundle

[FOSUserBundle](https://github.com/FriendsOfSymfony/FOSUserBundle) provides user management for Symfony project. Compatible with Doctrine ORM & ODM, and custom storages.

```bash
composer require friendsofsymfony/user-bundle
```

## jwt-authentication-bundle

[jwt-authentication-bundle](https://github.com/lexik/LexikJWTAuthenticationBundle) is JWT authentication for Symfony API.

Generate private, public keys at `config/jwt` folder:

```bash
openssl genrsa -out config/jwt/private.pem
openssl rsa -in config/jwt/private.pem -pubout > config/jwt/public.pem
```

## KnpMarkdownBundle

[KnpMarkdownBundle](https://github.com/KnpLabs/KnpMarkdownBundle) provides markdown conversion for a Symfony project.

## KnpPaginatorBundle

[KnpPaginatorBundle](https://github.com/KnpLabs/KnpPaginatorBundle) is SEO friendly Symfony paginator to sort and paginate:

```bash
composer require knplabs/knp-paginator-bundle
```

## KnpTimeBundle

[KnpTimeBundle](https://github.com/KnpLabs/KnpTimeBundle) provides helpers for time manipulation. 
It allows you to get string '2 days ago' from DateTime object.

## MakerBundle

[MakerBundle](https://github.com/symfony/maker-bundle) is the fastest way to generate the most common code you'll need in a Symfony app: commands, controllers, form classes, event subscribers and more.

```bash
composer require maker --dev
```

## SecurityBundle

[SecurityBundle](https://github.com/symfony/security-bundle) integrates the [Security component](https://github.com/symfony/security) in Symfony applications. All these options are configured under the `security` key in your application configuration:

```bash
composer require security
```

## SensioFrameworkExtraBundle

[SensioFrameworkExtraBundle](https://github.com/sensiolabs/SensioFrameworkExtraBundle) allows to configure controllers with annotations.

```bash
composer require sensio/framework-extra-bundle
```

## StofDoctrineExtensionsBundle 

[StofDoctrineExtensionsBundle](https://github.com/stof/StofDoctrineExtensionsBundle) 
provides integration for [DoctrineExtensions](https://github.com/Atlantic18/DoctrineExtensions) in your Symfony Project.

Allows to add `Timestampable` - updates date fields on create (created_at), update (updated_at) and even property change.

```bash
composer require stof/doctrine-extensions-bundle
```

Activate `timestampable` feature at `config\packages\stof_doctrine_extensions.yaml`:

```yaml
# Read the documentation: https://symfony.com/doc/current/bundles/StofDoctrineExtensionsBundle/index.html
# See the official DoctrineExtensions documentation for more details: https://github.com/doctrine-extensions/DoctrineExtensions/tree/main/doc
stof_doctrine_extensions:
    default_locale: en_US
    orm:
        default:
            timestampable: true
```

After this add `TimestampableEntity` trait to your entity:

```php
class Blog
{
    use TimestampableEntity;
    // ...
}
```
