# Entity

### Create entity / Edit entity

```bash
php bin/console make:entity
```

This command will create 2 files:

- src/Entity/Question.php
- src/Repository/QuestionRepository.php

### Generate migration

```bash
# This command reads env variables from Docker
symfony console make:migration

# This command doesn't have access to Docker's env vars
php bin/console make:migration
```

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

**Important:** If you use UUID for primary key don't forget to define correct return type for `getId` method from `?int` to `?string`:

```php
class Comment
{
    public function getId(): ?string
    {
        return $this->id;
    }
```
