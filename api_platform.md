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
```
