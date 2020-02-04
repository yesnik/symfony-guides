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
