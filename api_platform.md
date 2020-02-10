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

### Adding groups to fields

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
 *     collectionOperations={"get", "post"},
 *     itemOperations={
 *          "get"={},
 *          "put"
 *     },
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
```

Choose the exact properties you want with `?properties[]=title&properties[]=shortDescription`

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
 
