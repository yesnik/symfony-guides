# Doctrine

## Install doctrine

```
composer require doctrine
```

Edit `.env` to define `DATABASE_URL`:

```
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
```

## Annotations 

- Annotations *only support double quotes*. We cannot use single quotes there.
- Every annotation has a concrete PHP class behind it

### Annotations for entity's column

Let's suppose that we have class at `src/Entity/User.php`

```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Serializer\Annotation\Groups;

/**
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 */
class User implements UserInterface
{
    // We place annotation here
    private $some_column;
}
```

**primary key**

```php
/**
 * @ORM\Id()
 * @ORM\GeneratedValue()
 * @ORM\Column(type="integer")
 */
private $id;
```

**string - unique**

```php
/**
 * @ORM\Column(type="string", length=180, unique=true)
 * @Groups("main")
 */
private $email;
```

**string - allow null**

```php
/**
 * @ORM\Column(type="string", length=255, nullable=true)
 * @Groups("main")
 */
private $firstName;
```

**integer**

```php
/**
 * @ORM\Column(type="integer")
 */
private $likesCount = 0;
```

**boolean**

```php
/**
 * @ORM\Column(type="boolean")
 */
private $isDeleted = false;
```

For MySQL it'll create `TINYINT` data type.

**text**

```php
/**
 * @ORM\Column(type="text")
 */
private $content;
```

**datetime**

```php
/**
 * @ORM\Column(type="datetime")
 */
private $expiresAt;
```

**JSON column**

```php
/**
 * @ORM\Column(type="json")
 */
private $roles = [];
```

**relation - one to many**

User has many articles. At `Article` we have `author` field.

```php
/**
 * @ORM\OneToMany(targetEntity="App\Entity\Article", mappedBy="author")
 */
private $articles;
```

### Create entity (Entity create) / Add column / Add field

```
bin/console make:entity
```

Enter the name of new or existing entity. This command allows you to add new column.

If you define `Article` as name then this command will create the file `src/Entity/Article.php`:

```php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity(repositoryClass="App\Repository\ArticleRepository")
 */
class Article
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;

    /**
     * @ORM\Column(type="string", length=255)
     */
    private $title;

    /**
     * @ORM\Column(type="text", nullable=true)
     */
    private $content;

    /**
     * @ORM\Column(type="datetime", nullable=true)
     */
    private $publishedAt;

    public function getId(): ?int
    {
        return $this->id;
    }
    
    // ...
}
```

What makes this class special are the **annotations**! 
The `@ORM\Entity` above the class tells Doctrine that this is an entity that should be mapped to the database.

Above each property, we have some annotations that help doctrine know how to store that exact column.

### Drop column / remove column

Edit `src/Entity/Article.php`, remove property `author` and getter, setter methods - `getAuthor()`, `setAuthor()`.

Run command:

```
php bin/console make:migration
```
This command will create migration file `src/Migrations/Version20190206071030.php`.

### Alter column

Edit `src/Entity/User.php`, edit `nullable=true` to annotation of `$firstName` field.

Run command:

```
php bin/console make:migration
```

This command will create migration file `src/Migrations/Version20190214111526.php`.

### Create record

Edit `src/Controller/ArticleAdminController.php`:

```php
namespace App\Controller;

use App\Entity\Article;
use Doctrine\ORM\EntityManagerInterface;

class ArticleAdminController extends AbstractController
{
    /**
     * @Route("/admin/article/new")
     */
    public function new(EntityManagerInterface $em)
    {
        $article = new Article();
        $article->setTitle('Hello world')
                ->setSlug('hello-world');
        
        // It simply says that you would like to save this article
        $em->persist($article);
        
        // This command executes INSERT query
        $em->flush();

        return new Response(sprintf(
            'Hiya! New Article id: #%d slug: %s',
            $article->getId(),
            $article->getSlug()
        ));
    }
}
```

### Find records

Whenever you need to run a query, step one is to get that *entity's repository*. For example, when we generated the `Comment` class, the `make:entity` command also gave us a new `CommentRepository`.

**.findOneBy**

Edit `src/Controller/ArticleAdminController.php`:

```php
/** @var EntityManagerInterface $em */
$repository = $em->getRepository(Article::class);

$article = $repository->findOneBy(['slug' => $slug]);

if (!$article) {
    throw $this->createNotFoundException(sprintf('No article for slug "%s"', $slug));
}
```

**.findAll**

```php
$repository = $em->getRepository(Article::class);
$articles = $repository->findAll();
```

**.findBy**

This will sort records by `publishedAt` in descending order.

```php
$repository = $em->getRepository(Article::class);
$articles = $repository->findBy(['published' => '1'], ['publishedAt' => 'DESC']);
```
