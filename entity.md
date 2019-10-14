# Entity

### Trait TimestampableEntity

Edit file of entity:

```php
use Gedmo\Timestampable\Traits\TimestampableEntity;

class Tag
{
    use TimestampableEntity;
    // ...
}
```

### Use UUID as primary key

If we want to use entity's `id` in other applications we need to change the `id` field of the entity to use the [Doctrine UUID](https://www.doctrine-project.org/projects/doctrine-orm/en/2.6/reference/basic-mapping.html#identifier-generation-strategies) instead of the default auto incremental ID:

```php
// src/entity/Article.php

class Comment
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue(strategy="UUID")
     * @ORM\Column(type="guid", unique=true)
     */
    private $id;
```

`UUID` tells Doctrine to use the built-in Universally Unique Identifier generator. This strategy provides full portability.
