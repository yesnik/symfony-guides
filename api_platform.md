# API Platform

[API Platform's](https://api-platform.com/) philosophy: create resources, tweak configuration, let API Platform expose those resources as an API. Additionally it creates an OpenAPI specification (visit `/api/docs.json`).

Install Symfony [api pack](https://github.com/api-platform/api-pack) for API Platform:

```
composer require api
```

**Available URLs**

- Web interface: http://127.0.0.1:8000/api
- OpenAPI specification: http://127.0.0.1:8000/api/docs.json
- JSON-LD specification with Hydra: http://127.0.0.1:8000/api/docs.jsonld
- Entity in JSON format: http://127.0.0.1:8000/api/products.json
- Entity in JSON-LD format http://127.0.0.1:8000/api/products.jsonld

OpenAPI and JSON-LD documents do the same thing: they describe our API in a machine-readable format.

## Generate Entity

```bash
php bin/console make:entity
```

Answer `yes` to this question: *Mark this class as an API Platform resource (expose a CRUD API for it) (yes/no)*

This command will generate `/src/Entity/CheeseListing.php`.
Annotation `@ApiResource()` tells API Platform that you want to expose this class as an API:

```php
namespace App\Entity;

use ApiPlatform\Core\Annotation\ApiResource;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ApiResource()
 * @ORM\Entity(repositoryClass="App\Repository\CheeseListingRepository")
 */
class CheeseListing
{
    /**
     * @ORM\Id()
     * @ORM\GeneratedValue()
     * @ORM\Column(type="integer")
     */
    private $id;
}
```

### Generate User

```bash
php bin/console make:user
```

This command will create:

- src/Entity/User.php
- src/Repository/UserRepository.php


## Operations

API Platform supports 2 types of operations: collection operations, item operations.

Edit `/src/Entity/CheeseListing.php`:

```php
/**
 * @ApiResource(
 *     collectionOperations={"get", "post"},
 *     itemOperations={"get", "put", "delete"}
 * )
 * @ORM\Entity(repositoryClass="App\Repository\CheeseListingRepository")
 */
class CheeseListing
{
    // ...
```

### Customize operation's URL

Each operation generates a route, and API Platform gives you full control over how that route looks.

```php
/**
 * @ApiResource(
 *     collectionOperations={"get", "post"},
 *     itemOperations={
 *          "get"={"path"="/i_love_cheeses/{id}"},
 *          "put"
 *     },
 *     shortName="cheeses"
 * )
 */
class CheeseListing { // ... }
```
This will change GET route to `/api/i_love_cheeses/{id}`.

## Modify search query

If we need to restrict the items returned by the API, create a service that implements:
- `QueryCollectionExtensionInterface` if we need to control the Doctrine query used for collections
- `QueryItemExtensionInterface` if we need to control items.

Create file `src/Api/FilterPublishedCommentQueryExtension.php`:

```php
namespace App\Api;

use ApiPlatform\Core\Bridge\Doctrine\Orm\Extension\QueryCollectionExtensionInterface;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Extension\QueryItemExtensionInterface;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Util\QueryNameGeneratorInterface;
use App\Entity\Comment;
use Doctrine\ORM\QueryBuilder;

class FilterPublishedCommentQueryExtension implements QueryCollectionExtensionInterface, QueryItemExtensionInterface
{
    public function applyToCollection(QueryBuilder $qb, QueryNameGeneratorInterface $queryNameGenerator, string $resourceClass, string $operationName = null)
    {
        if (Comment::class === $resourceClass) {
            $qb->andWhere(sprintf("%s.state = 'published'", $qb->getRootAliases()[0]));
        }
    }

    public function applyToItem(QueryBuilder $qb, QueryNameGeneratorInterface $queryNameGenerator, string $resourceClass, array $identifiers, string $operationName = null, array $context = [])
    {
        if (Comment::class === $resourceClass) {
            $qb->andWhere(sprintf("%s.state = 'published'", $qb->getRootAliases()[0]));
        }
    }
}
```
The query extension class applies its logic only for the `Comment` resource and modify the Doctrine query builder to only consider comments in the published state.

## Groups

Use annotations to edit groups:

- `normalizationContext` - for read
- `denormalizationContext` - for write

These groups helps us to expose to our API only the fields that we want.

```php
use ApiPlatform\Core\Annotation\ApiResource;
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Serializer\Annotation\Groups;
/**
 * @ApiResource(
 *     shortName="cheeses",
 *     normalizationContext={"groups"={"cheese_listing:read"}, "swagger_definition_name"="Read"},
 *     denormalizationContext={"groups"={"cheese_listing:write"}, "swagger_definition_name"="Write"}
 * )
 * @ORM\Entity(repositoryClass="App\Repository\CheeseListingRepository")
 */
class CheeseListing
{
    /**
     * @ORM\Column(type="string", length=255)
     * @Groups({"cheese_listing:read", "cheese_listing:write"})
     */
    private $title;

    /**
     * @ORM\Column(type="text")
     * @Groups({"cheese_listing:read"})
     */
    private $description;
    
    /**
     * The description of the cheese as raw text.
     *
     * @Groups("cheese_listing:write")
     */
    public function setTextDescription(string $description): self
    {
        $this->description = nl2br($description);
        return $this;
    }
    
    /**
     * How long ago in text that this cheese listing was added.
     *
     * @Groups("cheese_listing:read")
     */
    public function getCreatedAtAgo(): string
    {
        return Carbon::instance($this->getCreatedAt())->diffForHumans();
    }
```

### Change items per page

```php
/**
 * @ApiResource(
 *     attributes={
 *         "pagination_items_per_page"=10
 *     }
 * )
 */
class CheeseListing {}
```

### Change name of property

In write operations `textDescription` will be named `description`, thanks to annotation `@SerializedName`.

```php
use Symfony\Component\Serializer\Annotation\SerializedName;

class CheeseListing
{
    /**
     * The description of the cheese as raw text.
     *
     * @SerializedName("description")
     * @Groups("cheese_listing:write")
     */
    public function setTextDescription(string $description): self
    {
        $this->description = nl2br($description);
        return $this;
    }
```

## Filters

### BooleanFilter

```php
use ApiPlatform\Core\Annotation\ApiFilter;
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\BooleanFilter;

/**
 * @ApiFilter(BooleanFilter::class, properties={"isPublished"})
 */
class CheeseListing {}
```

Request:

```bash
curl "http://127.0.0.1:8000/api/cheeses?isPublished=false" -H "accept: application/ld+json"
```

### SearchFilter

```php
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\SearchFilter;

/**
 * @ApiFilter(SearchFilter::class, properties={"title": "partial"})
 */
class CheeseListing {}
```
Request:

```bash
curl "http://127.0.0.1:8000/api/cheeses?title=Hello" -H "accept: application/ld+json"
```

**Use conditions on relation**

```php
/**
 * @ApiFilter(SearchFilter::class, properties={
 *     "title": "partial",
 *     "description": "partial",
 *     "owner": "exact",
 *     "owner.username": "partial"
 * })
 */
class CheeseListing {}
```
Requests:

- http://127.0.0.1:8000/api/cheeses.json?owner.username=sara
- http://127.0.0.1:8000/api/cheeses.json?owner[]=%2Fapi%2Fusers%2F4 (param `owner` accepts IRI value: `/api/users/4`)

### RangeFilter

```php
use ApiPlatform\Core\Bridge\Doctrine\Orm\Filter\RangeFilter;

/**
 * @ApiFilter(RangeFilter::class, properties={"price"})
 */
class CheeseListing {}
```

Request:

```bash
curl -X GET "http://127.0.0.1:8000/api/cheeses?price[gt]=1000" -H "accept: application/ld+json"
```

### PropertyFilter

```php
use ApiPlatform\Core\Serializer\Filter\PropertyFilter;

/**
 * @ApiResource()
 * @ApiFilter(PropertyFilter::class)
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 */
class User implements UserInterface {}
```

Choose the exact properties you want with `?properties[]=title&properties[]=shortDescription`.

Also it's possible to select attributes of related entity: 
`/api/users/2.jsonld?properties[]=email&properties[cheeseListings][]=title&properties[cheeseListings][]=price`

## Formats

### Define formats for entity

```php
/**
 * @ApiResource(
 *     attributes={
 *         "formats"={"jsonld", "json", "html", "jsonhal", "csv"={"text/csv"}}
 *     }
 * )
 */
class CheeseListing {}
```

### Define formats globally

Display available config options for API Platform:

```bash
php bin/console debug:config api_platform
```

Look at the `formats` section in the output:

```yaml
api_platform:
    formats:
        jsonld:
            mime_types:
                - application/ld+json
        json:
            mime_types:
                - application/json
        html:
            mime_types:
                - text/html
        jsonhal:
            mime_types:
                - application/hal+json
```

Copy these lines to `config/packages/api_platform.yaml`.

## Validation

Validation in API Platform works exactly like validation in every Symfony app.
 
Edit file:
 
```php
use Symfony\Component\Validator\Constraints as Assert;

/**
 * @ORM\Entity(repositoryClass="App\Repository\CheeseListingRepository")
 */
class CheeseListing
{
    /**
     * @ORM\Column(type="string", length=255)
     * @Assert\NotBlank()
     * @Assert\Length(
     *     min=2,
     *     max=50,
     *     maxMessage="Describe your cheese in 50 chars or less"
     * )
     */
    private $title;
    
 ```
 
If we don't define `groups` param on a constraint, the validator puts this constraint into `Default` group. 
By default the validator executes constraints in this `Default` group.

Suppose that we don't want it to include `@Assert\NotBlank` validator into `PUT` operation. But we want it in the `POST` operation. It's possible to specify `validation_groups` for each method in operaion:

```php
/**
 * @ApiResource(
 *     collectionOperations={
 *          "post"={
 *              "security"="is_granted('IS_AUTHENTICATED_ANONYMOUSLY')",
 *              "validation_groups"={"Default", "create"}
 *          },
 *     },
 * )
 */
class User implements UserInterface {
    /**
     * @Groups("user:write")
     * @Assert\NotBlank(groups={"create"})
     * @SerializedName("password")
     */
    private $plainPassword;
    
```

The `POST` operation will execute all validation constraints in `Default` and `create` groups.
 
 ## Relations
 
 Suppose that CheeseListing belongs to User:
 
 ```php
 /**
 * @ApiResource(
 *     normalizationContext={"groups"={"cheese_listing:read"}, "swagger_definition_name"="Read"},
 *     denormalizationContext={"groups"={"cheese_listing:write"}, "swagger_definition_name"="Write"},
 * )
 * @ORM\Entity(repositoryClass="App\Repository\CheeseListingRepository")
 */
class CheeseListing {
    /**
     * @ORM\ManyToOne(targetEntity="App\Entity\User", inversedBy="cheeseListings")
     * @ORM\JoinColumn(nullable=false)
     * @Groups({"cheese_listing:read", "cheese_listing:write"})
     */
    private $owner;
    
 ```
 
 In POST request we need to provide `IRI` value for `owner` field:
 
```json
{
  "title": "Cheese",
  "price": 150,
  "owner": "/api/users/1",
  "description": "Nice cheese"
}
```

### Embed relation data in response

Add `"cheese_listing:read"` group to `User` entity:

```php
class User implements UserInterface
{
    /**
     * @ORM\Column(type="string", length=180, unique=true)
     * @Groups({"user:read", "user:write", "cheese_listing:read"})
     */
    private $email;

    /**
     * @ORM\Column(type="string", length=255, unique=true)
     * @Groups({"user:read", "user:write", "cheese_listing:read"})
     */
    private $username;
```

The serializer knows to serialize all fields in the `cheese_listing:read` group. 
It first looks at `CheeseListing` and finds `title`, `description` and `owner`. Then it keeps going and, inside of `User`, finds that group on `email` and `username`.

Result:

```json
{
  "title": "Cheese",
  "description": "Nice cheese 1",
  "price": 201,
  "owner": {
    "email": "kenny@mail.ru",
    "username": "kenny"
  }
}
```

If you get back an object, it will have `@id`, `@type` and other data properties. If you get back a string, you know it's an IRI that you can use to go get the real data.

### Embed relation's data only for GET

Use a specific naming convention for this *operation-specific group*:

```
[entity name] : [item | collection] : [get | post | put | delete ]
```

Add group `cheese_listing:item:get` to `normalization_context` param:

```php
/**
 * @ApiResource(
 *     itemOperations={
 *          "get"={
 *              "normalization_context"={
 *                  "groups"={
 *                      "cheese_listing:read",
 *                      "cheese_listing:item:get"
 *                  }
 *              },
 *          },
 *          "put"
 *     },
 * )
 * @ORM\Entity(repositoryClass="App\Repository\CheeseListingRepository")
 */
class CheeseListing {}
```

Add this group to related entity:

```php
class User implements UserInterface
{
    /**
     * @ORM\Column(type="string", length=180, unique=true)
     * @Groups({
     *     "user:read", "user:write", "cheese_listing:item:get"
     * })
     */
    private $email;
```

### Update related entity

If we want to update related entity we need to provide `@id`:

```json
{
  "owner": {
    "@id": "/api/users/2",
    "username": "Leo"
  }
}
```

Otherwise we'll get an error: *A new entity was found through the relationship 'App\\Entity\\CheeseListing#owner' that was not configured to cascade persist operations for entity: App\\Entity\\User@0000...*

### Validate related entity

But when we're doing embedded object updates, we do want validation to continue down into this object. To force that, above the `owner` property, add `@Assert\Valid()`.

```php
class CheeseListing
{
    /**
     * @ORM\ManyToOne(targetEntity="App\Entity\User", inversedBy="cheeseListings")
     * @Assert\Valid()
     */
    private $owner;
```

When the validator processes the `User` object, it doesn't automatically cascade down into the `cheeseListings` array and also validate those objects. We can force that by adding `@Assert\Valid()`:

```php
class User implements UserInterface
{
    /**
     * @ORM\OneToMany(targetEntity="App\Entity\CheeseListing", mappedBy="owner", 
     *                orphanRemoval=false, cascade={"persist"})
     * @Groups({"user:read", "user:write"})
     * @Assert\Valid()
     */
    private $cheeseListings;
```

### Create User and assign it to existing entity

Add group `user:write` to `$cheeseListings` property.

```php
class User implements UserInterface
{
    /**
     * @ORM\OneToMany(targetEntity="App\Entity\CheeseListing", mappedBy="owner", orphanRemoval=false)
     * @Groups({"user:read", "user:write"})
     */
    private $cheeseListings;
```

This will allow us to make request `POST /api/users`:

```json
{
  "email": "lora@mail.ru",
  "password": "123",
  "username": "lora",
  "cheeseListings": [
    "/api/cheeses/1"
  ]
}
```

### Created main and related entity

Sometimes we can get an error: *Nested documents for attribute "cheeseListings" are not allowed. Use IRIs instead*.
It means that we need to add group `user:write` to CheeseListing's attributes:

```php
class CheeseListing
{
    /**
     * @ORM\Column(type="string", length=255)
     * @Groups({
     *     "cheese_listing:read", 
     *     "cheese_listing:write", 
     *     "user:read",
     *     "user:write"
     * })
     */
    private $title;

    /**
     * The price of this delicious cheese in cents.
     *
     * @ORM\Column(type="integer")
     * @Groups({
     *     "cheese_listing:read", 
     *     "cheese_listing:write", 
     *     "user:read",
     *     "user:write"
     * })
     */
    private $price;

    /**
     * @SerializedName("description")
     * @Groups({"cheese_listing:write", "user:write"})
     */
    public function setTextDescription(string $description): self
    {
        $this->description = nl2br($description);
        return $this;
    }
```

Also we can get this error: *A new entity was found through the relationship 'App\\Entity\\User#cheeseListings' that was not configured to cascade persist operations for entity: App\\Entity\\CheeseListing@000*.

This error tells us that API Platform is creating a new `CheeseListing` and it is setting it onto the `cheeseListings` property of the new `User`. But nothing ever calls `$entityManager->persist()` on that new `CheeseListing`, which is why Doctrine isn't sure what to do when trying to save the `User`.

To fix this error add `cascade={"persist"}`:

```php
class User implements UserInterface
{
    /**
     * @ORM\OneToMany(targetEntity="App\Entity\CheeseListing", mappedBy="owner", orphanRemoval=false, cascade={"persist"})
     * @Groups({"user:read", "user:write"})
     */
    private $cheeseListings;
```

### Subresources

If you want to make the link `/api/users/4/cheese_listings` work use `@ApiSubresource()`:

```php
use ApiPlatform\Core\Annotation\ApiSubresource;

class User implements UserInterface
{
    /**
     * @ORM\OneToMany(targetEntity="App\Entity\CheeseListing", mappedBy="owner", orphanRemoval=true, cascade={"persist"})
     * @Groups({"user:read", "user:write"})
     * @Assert\Valid()
     * @ApiSubresource()
     */
    private $cheeseListings;
```

## Normalizer

Create normalizer:

```bash
php bin/console make:serializer:normalizer
```

This command will create file `src/Serializer/Normalizer/UserNormalizer.php`.

When an object is being transformed into JSON (or another format), it goes through these steps: 

1. *Normalizer* transforms the object into an array. 
2. *Encoder* transforms that array into JSON (or another format).

The serializer has many normalizers. When it needs to normalize something, it loops over all the normalizers, calls `supportsNormalization()` and passes us the data that it needs to normalize. 
If we return `true` from `supportsNormalization()`, the serializer will call `normalize()` method. 


## Authorization

When a request comes in, API Platform goes through three steps in a specific order:

1. It deserializes the JSON and updates the CheeseListing object
2. It applies `security` param 
3. It executes validation rules.

### Protect operation

Edit `src/Entity/CheeseListing.php`:

```php
/**
 * @ApiResource(
 *     collectionOperations={
 *          "get",
 *          "post"={"security"="is_granted('ROLE_USER')"}
 *     },
 *     itemOperations={
 *          "put"={
 *              "security"="is_granted('ROLE_USER') and object.getOwner() == user",
 *              "security_message"="Only the creator can edit a cheese listing"
 *          },
 *          "delete"={"access_control"="is_granted('ROLE_ADMIN')"}
 *     },
 *     ...
 * )
 */
class CheeseListing { }
```

If you're not logged in then you can't make POST request to create `CheeseListing`.

Edit `src/Entity/User.php`:

```php
/**
 * @ApiResource(
 *     collectionOperations={
 *          "get"={"access_control"="is_granted('ROLE_USER')"},
 *          "post"
 *     },
 *     itemOperations={
 *          "get"={"access_control"="is_granted('ROLE_USER')"},
 *          "put"={"access_control"="is_granted('ROLE_USER') and object == user"},
 *          "delete"={"access_control"="is_granted('ROLE_ADMIN')"}
 *     },
 *     normalizationContext={"groups"={"user:read"}},
 *     denormalizationContext={"groups"={"user:write"}},
 * )
 * @UniqueEntity(fields={"username"})
 * @UniqueEntity(fields={"email"})
 * @ApiFilter(PropertyFilter::class)
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 */
class User implements UserInterface {}
```

## Testing API

### Api test example

File `tests/Functional/CheeseListingResourceTest.php`:

```php
namespace App\Tests\Functional;

use ApiPlatform\Core\Bridge\Symfony\Bundle\Test\ApiTestCase;
use App\Entity\User;
use Hautelook\AliceBundle\PhpUnit\ReloadDatabaseTrait;

class CheeseListingResourceTest extends ApiTestCase
{
    use ReloadDatabaseTrait;

    public function testCreateCheeseListing()
    {
        $client = self::createClient();

        $client->request('POST', '/api/cheeses', [
            'headers' => ['Content-Type' => 'application/json'],
            'json' => [],
        ]);

        $this->assertResponseStatusCodeSame(401);


        $user = new User();
        $user->setEmail('cheeseplease@example.com');
        $user->setUsername('cheeseplease');
        $user->setPassword('$2y$13$hle7zuQLVA.9csajbOTET.bhwp8CkaoyjY9tyGdJ2Nhu.stdCO9ES');
        $em = self::$container->get('doctrine')->getManager();
        $em->persist($user);
        $em->flush();

        $client->request('POST', '/login', [
            'headers' => ['Content-Type' => 'application/json'],
            'json' => [
                'email' => 'cheeseplease@example.com',
                'password' => 'foo'
            ],
        ]);
        $this->assertResponseStatusCodeSame(204);
    }
}
```

To clear DB we use here [AliceBundle](https://github.com/hautelook/AliceBundle).

Always start with `self::createClient()`. It creates a `$client` object that will help us make requests into our API. 
Also it *boots Symfony's container*, which is what gives us access to the entity manager and all other services. 

