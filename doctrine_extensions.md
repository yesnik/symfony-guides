# Doctrine Extensions

[StofDoctrineExtensionsBundle](https://github.com/stof/StofDoctrineExtensionsBundle) is a layer around another 
library called [DoctrineExtensions](https://github.com/Atlantic18/DoctrineExtensions), which 
can add some features to our entities.

```
composer require stof/doctrine-extensions-bundle
```

To activate desired feature edit `config/packages/stof_doctrine_extensions.yaml`:

```yaml
stof_doctrine_extensions:
    default_locale: en_US
    orm:
        default:
            sluggable: true
            timestampable: true
```

## Sluggable

It urlizes your specified fields into single unique slug.

Let's apply `@Gedmo\Slug` annotation to the field where we want store slug.

Edit `src/Entity/Question.php`:

```php
use Gedmo\Mapping\Annotation as Gedmo;

class Question
{
    /**
     * @ORM\Column(type="string", length=100, unique=true)
     * @Gedmo\Slug(fields={"name"})
     */
    private $slug;
    
    /**
     * @ORM\Column(type="string", length=255)
     */
    private $name;
}
```

## Timestampable

Updates date fields on create, update and even property change.

Edit `src/Entity/Question.php`:

```php
use Gedmo\Timestampable\Traits\TimestampableEntity;

class Question
{
    use TimestampableEntity;
}
```

## Loggable 

It helps tracking changes and history of objects, also supports version management.
