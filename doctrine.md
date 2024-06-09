# Doctrine

[Doctrine](https://www.doctrine-project.org/) doesn't want you to think about tables or columns, it wants you to think about classes and properties.

## Install Doctrine

```bash
composer require orm
```

This command will require packages from [orm-pack](https://github.com/symfony/orm-pack). A pack is just a shortcut to install several packages.

Edit `.env` to define `DATABASE_URL`:

```.env
# MySQL
DATABASE_URL=mysql://db_user:db_password@127.0.0.1:3306/db_name
# PostgreSQL
DATABASE_URL="postgresql://db_user:db_password@127.0.0.1:5432/db_name?serverVersion=16&charset=utf8"
```

## Attributes 

- Before PHP 8 we had to use annotations that *only support double quotes*. We can't use single quotes there.
- Every attribute (annotation) has a concrete PHP class behind it

### Attributes for entity's column

Let's suppose that we have a class at `src/Entity/User.php`

```php
namespace App\Entity;

use Doctrine\Common\Collections\ArrayCollection;
use Doctrine\Common\Collections\Collection;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Serializer\Annotation\Groups;
use App\Repository\UserRepository;

#[ORM\Entity(repositoryClass: UserRepository::class)]
class User implements UserInterface
{
    // We place PHP attribute (or annotation) here
    private $some_column;
}
```

#### primary key

```php
#[ORM\Id]
#[ORM\GeneratedValue]
#[ORM\Column]
private ?int $id = null;
```

#### string - not null

```php
#[ORM\Column(length: 255)]
private ?string $title = null;
```

#### string - unique

```php
#[ORM\Column(length: 180, unique: true)]
private ?string $email = null;
```

#### string - allow null

```php
#[ORM\Column(length: 255, nullable: true)]
private ?string $firstName = null;
```

#### string - default value

```php
#[ORM\Column(length: 255, options: ['default' => 'active'])]
private ?string $status = null;
```

#### integer

```php
#[ORM\Column(type: 'integer')
private $likesCount = 0;
```

#### boolean

```php
#[ORM\Column(type: 'boolean')]
private $isDeleted = false;
```

For MySQL it'll create `TINYINT` data type.

#### text

```php
#[ORM\Column(type: 'text')]
private $content;
```

#### datetime

```php
#[ORM\Column(type: 'datetime')]
private $expiresAt;
```

#### JSON column

```php
#[ORM\Column(type: 'json')]
private $roles = [];
```

#### relation - one to many

One User can have many articles. At `Article` we have `author` field.

```php
#[ORM\OneToMany(targetEntity= Article::class, mappedBy: 'author')]
private Collection $articles;
```

### UniqueEntity constraint

If you want to validate that the value of an entity property is unique among all entities of the same type (e.g. the registration email of all users) use the [UniqueEntity](https://symfony.com/doc/current/reference/constraints/UniqueEntity.html) constraint.

```php
use Doctrine\ORM\Mapping as ORM;
use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;

/**
 * @UniqueEntity("username")
 * @UniqueEntity("email")
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 */
class User implements UserInterface { }
```

In this example we need to require two fields to be individually unique (e.g. a unique `email` and a unique `username`), so we use two `UniqueEntity` entries, each with a single field.

## Entity

### Create entity / Add column / Add field

```bash
bin/console make:entity
```

Enter the name of a new or existing entity. This command also allows you to add a new column.

If you define `Article` as name then this command will create the file `src/Entity/Article.php`:

```php
namespace App\Entity;

use App\Repository\ArticleRepository;
use Doctrine\DBAL\Types\Types;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ArticleRepository::class)]
class Article
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column]
    private ?int $id = null;

    #[ORM\Column(length: 255)]
    private ?string $title = null;

    #[ORM\Column(type: Types::TEXT, nullable: true)]
    private ?string $content = null;

    #[ORM\Column(type: Types::DATETIME_MUTABLE, nullable: true)]
    private ?\DateTimeInterface $publishedAt = null;

    public function getId(): ?int
    {
        return $this->id;
    }
    // ...
}
```

The `ORM\Entity` above the class tells Doctrine that this is an entity that should be mapped to the database.

Above each property, we have some PHP attributes that help Doctrine know how to store that exact column.

### Drop column / remove column

Edit `src/Entity/Article.php`, remove property `author` and getter, setter methods - `getAuthor()`, `setAuthor()`.

Generate migration file: `php bin/console make:migration`

### Alter column

Edit `src/Entity/User.php`, edit `nullable=true` to annotation of `$firstName` field.

Generate migration file: `php bin/console make:migration`

### Create record in a Controller

Edit `src/Controller/ArticleAdminController.php`:

```php
namespace App\Controller;

use App\Entity\Article;
use Doctrine\ORM\EntityManagerInterface;

class ArticleAdminController extends AbstractController
{
    #[Route('/admin/article/new')]
    public function new(EntityManagerInterface $em)
    {
        $article = (new Article())
            ->setTitle('Hello world')
            ->setSlug('hello-world');
        
        // It simply says that we want to save this article
        $em->persist($article);
        
        // It executes INSERT query
        $em->flush();

        return new Response(sprintf(
            'Hiya! New Article id: #%d slug: %s',
            $article->getId(),
            $article->getSlug()
        ));
    }
}
```

## `Doctrine\ORM\EntityRepository` methods

To run a query we need to get entity's repository. For example, `App\Repository\ArticleRepository`. If a Doctrine can't find a record it will return `null`.

#### .find

Finds an entity by its primary key / identifier.

```php
$article = $articleRepository->find(2);
```

#### .findAll

```php
$articles = $articleRepository->findAll();
```

#### .findBy

Find records by params and sort them by `id` in descending order.

```php
$articles = $articleRepository->findBy(['published' => '1'], ['id' => 'DESC']);

// Find 5 first records
$articles = $articleRepository->findBy(['published' => '1'], ['id' => 'DESC'], 5);

// Find 5 first records, offset 10
$articles = $articleRepository->findBy(['published' => '1'], ['id' => 'DESC'], 5, 10);
```

#### .findOneBy

Finds a single entity by params.

```php
$article = $articleRepository->findOneBy(['slug' => $slug]);
```

#### .count

Counts entities by a set of criteria.

```php
$count = $articleRepository->count(['published' => '1']);
```

### Relations

Let's add relation *Comment -> Article*:

```
php bin/console make:entity

> Comment

Your entity already exists! So let's add some new fields!

New property name (press <return> to stop adding fields):
> author

Field type (enter ? to see all types) [string]:
> relation

What class should this entity be related to?:
> Article

Relation type? [ManyToOne, OneToMany, ManyToMany, OneToOne]:
> ManyToOne
```

This command will update files: `src/Entity/Article.php`, `src/Entity/Comment.php`.
Look at `src/Entity/Article.php`:

```php
/**
 * @ORM\Entity(repositoryClass="App\Repository\ArticleRepository")
 */
class Article
{
    // ...
    /**
     * @ORM\OneToMany(targetEntity="App\Entity\Comment", mappedBy="article")
     */
    private $comments;

    public function __construct()
    {
        $this->comments = new ArrayCollection();
    }
    // ...
}
```

Look at `src/Entity/Comment.php`:

```php
/**
 * @ORM\Entity(repositoryClass="App\Repository\CommentRepository")
 */
class Comment
{
    /**
     * @ORM\ManyToOne(targetEntity="App\Entity\Article", inversedBy="comments")
     * @ORM\JoinColumn(nullable=false)
     */
    private $article;
}
```

And, it's easy to remember: the *owning side* is the side where the actual column appears in the database. 
Because the `comment` table will have the `article_id` column, the `Comment.article` property is the owning side. 
And so, `Article.comments` is the *inverse side*.

The reason this is so important is that, when you relate two entities together and save, 
Doctrine only looks at the *owning side* of the relationship to figure out what to persist to the database. 
When Doctrine saves, it looks at the `article` property on `Comment` - the owning side - sees that it is `null`, and tries to save the `Comment` with no `Article`!

The *owning side* is the only side where the data matters when saving. 
In fact, the entire purpose of the inverse side of the relationship is just convenience! 
It only exists because it's useful to be able to say `$article->getComments()`.

### Sort in DESC order

Add annotation `@ORM\OrderBy({"createdAt" = "DESC"})` to `src/Entity/Article.php`:

```php
class Article {
    /**
     * @ORM\OneToMany(targetEntity="App\Entity\Comment", mappedBy="article")
     * @ORM\OrderBy({"createdAt" = "DESC"})
     */
    private $comments;
    // ...
}
```

### Doctrine Lifecycle Callbacks

[Lifecycle callbacks](https://symfony.com/doc/7.2/doctrine/events.html#doctrine-lifecycle-callbacks) are defined as public methods inside the entity you want to modify. For example, suppose you want to set a createdAt date column to the current date, but only when the entity is first persisted (i.e. inserted). To do so, define a callback for the prePersist Doctrine event:

```php
// src/Entity/Product.php
namespace App\Entity;

use Doctrine\ORM\Mapping as ORM;

// Don't forget to add #[ORM\HasLifecycleCallbacks]
// to the class of the entity where you define the callback

#[ORM\Entity]
#[ORM\HasLifecycleCallbacks]
class Product
{
    // ...

    #[ORM\PrePersist]
    public function setCreatedAtValue(): void
    {
        $this->createdAt = new \DateTimeImmutable();
    }
}
```

## Query Builder

Doctrine also provides a [Query Builder](https://www.doctrine-project.org/projects/doctrine-orm/en/current/reference/query-builder.html), an object-oriented way to write queries. 
It is recommended to use this when queries are built dynamically (i.e. based on PHP conditions).

Edit `src/Repository/ArticleRepository.php`:

```php
class ArticleRepository extends ServiceEntityRepository
{
    public function findAllPublishedOrderedByNewest()
    {
        return $this->createQueryBuilder('a')
            ->andWhere('a.publishedAt IS NOT NULL')
            ->orderBy('a.publishedAt', 'DESC')
            ->getQuery()
            ->getResult();
    }
}
```

### Use filter

Filter file `src\Filter\BlogFilter.php`:

```php
class BlogFilter
{
    private ?string $title = null;

    public function getTitle(): ?string
    {
        return $this->title;
    }

    public function setTitle(?string $title): BlogFilter
    {
        $this->title = $title;
        return $this;
    }
}
```

File `src\Repository\BlogRepository.php`:

```php
class BlogRepository extends ServiceEntityRepository
{
    public function findByBlogFilter(BlogFilter $blogFilter): array
    {
        $qb = $this->createQueryBuilder('b');

        if ($blogFilter->getTitle()) {
            $qb->andWhere('b.title LIKE :title')
                ->setParameter('title', '%' . $blogFilter->getTitle() . '%')
                ->orderBy('b.title');
        }

        return $qb->getQuery()->getResult();
    }
```

### Join article to comment

If your page has a lot of queries because Doctrine is making extra queries across a relationship, just join over that relationship and use `addSelect()` to fetch all the data you need at once.

```php
class CommentRepository extends ServiceEntityRepository
{
    public function findAllWithSearch(?string $term)
    {
        $qb = $this->createQueryBuilder('c')
            ->innerJoin('c.article', 'a')
            ->addSelect('a');
            
        if ($term) {
            $qb->andWhere('c.content LIKE :term OR c.authorName LIKE :term OR a.title LIKE :term')
                ->setParameter('term', '%' . $term . '%')
            ;
        }
        
        return $qb
            ->orderBy('c.createdAt', 'DESC')
            ->getQuery()
            ->getResult()
        ;
    }
```

## Doctrine config

Let's see the meaning of some keys at `config/packages/doctrine.yaml`:

**Server version**

```
doctrine:
    dbal:
        server_version: '5.7'
```

This tells Doctrine that when it interacts with the database, it should expect that our DB has all the features supported by MySQL 5.7, including that native *JSON* column type.
