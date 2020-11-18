# Doctrine

Doctrine doesn't want you to think about tables or columns, it wants you to think about classes and properties.

## Install doctrine

```
composer require orm
```
This command will require packages from [orm-pack](https://github.com/symfony/orm-pack). A pack is just a shortcut to install several packages.

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

## Query Builder

To write a custom query, you can either create a *DQL* (Doctrine query language) string directly, 
or you can use the *query builder* that helps create a DQL string.

Method returns a array of Comment objects. It does not, for example, now return Comment and Article objects.

Instead, Doctrine takes that extra article data and stores it in the background for later. 
But, the new `addSelect()` does not affect the return value. That's way different than using raw SQL.

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
    // ...
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
