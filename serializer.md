# Symfony Serializer Component

[Docs](https://symfony.com/doc/current/components/serializer.html)

 An *array* is used as an intermediary between objects and serialized contents. 
 This way, encoders will only deal with turning specific formats into arrays and vice versa. 
 The same way, *Normalizers* will deal with turning specific objects into arrays and vice versa.
 
 ```bash
 composer require symfony/serializer
 ```

## Serialize Entity

In a controller:

```php
class BlogController extends AbstractController
{
    #[Route('/api/blog', name: 'app_api_blog')]
    public function index(BlogRepository $blogRepository): Response
    {
        $blogs = $blogRepository->getBlogs();

        return $this->json($blogs);
    }
}
```

### Fix circular reference error

Modify `config/services.yaml`:

```yml
services:
    app.normalizer.object_normalizer:
        class: Symfony\Component\Serializer\Normalizer\ObjectNormalizer
        tags: ['serializer.normalizer']
        arguments:
            $defaultContext:
                circular_reference_handler: '@App\Serializer\CircularReferenceHandler'
                ignored_attributes: ['createdOn', 'updatedOn']
```

In `ignored_attributes` option we define what entity's fields should not be included in the output.

Add a handler `src\Serializer\CircularReferenceHandler.php`:

```php
namespace App\Serializer;

class CircularReferenceHandler
{
    public function __invoke($object)
    {
        return $object->getId();
    }
}
```

### Ignore entity's field

#### Via `Ignore`

File `src/Entity/Blog.php`:

```php
namespace App\Entity;

use Symfony\Component\Serializer\Attribute\Ignore;

#[ORM\Entity(repositoryClass: BlogRepository::class)]
#[ORM\HasLifecycleCallbacks]
class Blog
{
    #[ORM\ManyToOne(cascade:['persist'], inversedBy: 'blogs')]
    #[Ignore]
    private ?User $user = null;
```

#### Via `context` param for `json()` method

In a controller:

```php
  return $this->json($blogs, context: [AbstractNormalizer::IGNORED_ATTRIBUTES => [
          'category',
          'createdAt',
      ]
  ]);
```

#### Via `Groups`

Entity:

```php
class Blog
{
    #[Groups(['groupOne'])]
    #[Assert\NotBlank]
    #[ORM\Column(length: 255)]
    private ?string $title = null;
```

Controller:

```php
    return $this->json($blogs, context: [
        AbstractNormalizer::GROUPS => ['groupOne'],
    ]);
```

Fields marked with `groupOne` will be in the response.

### Format attribute

Modify `config/services.yaml` - define callback for the field `updatedAt`:

```yml
services:
    app.normalizer.object_normalizer:
        class: Symfony\Component\Serializer\Normalizer\ObjectNormalizer
        tags: ['serializer.normalizer']
        arguments:
            $defaultContext:
                circular_reference_handler: '@App\Serializer\CircularReferenceHandler'
                ignored_attributes: ['createdOn', 'updatedOn']
                callbacks: {
                    'updatedAt': '@App\Serializer\UpdatedAtCallback',
                }
```

File `src/Serializer/UpdatedAtCallback.php`

```php
class UpdatedAtCallback
{
    public function __invoke(null|string|DateTimeInterface $innerObject): DateTimeInterface|string|null
    {
        if ($innerObject === null) {
            return null;
        }

        if (!($innerObject instanceof DateTimeInterface)) {
            return $innerObject;
        }

        return $innerObject->format('Y-m-d H:i:s');
    }
}
```

## Serialize Object
 
```php
use Symfony\Component\Serializer\Encoder\JsonEncoder;
use Symfony\Component\Serializer\Encoder\XmlEncoder;
use Symfony\Component\Serializer\Normalizer\ObjectNormalizer;
use Symfony\Component\Serializer\Serializer;

$encoders = [new XmlEncoder(), new JsonEncoder()];
$normalizers = [new ObjectNormalizer()];

$serializer = new Serializer($normalizers, $encoders);

use App\Model\Person;

$person = new Person();
$person->setName('foo');
$person->setAge(99);
$person->setSportsperson(false);

$jsonContent = $serializer->serialize($person, 'json');

// $jsonContent contains {"name":"foo","age":99,"sportsperson":false,"createdAt":null}

echo $jsonContent; // or return it in a Response
```

The first parameter of the `serialize()` is the object to be serialized and the second is used to choose the proper encoder, 
in this case `JsonEncoder`.

## Deserializing an Object

The information of the `Person` class is encoded in XML format:

```php
use App\Model\Person;

$data = <<<EOF
<person>
    <name>foo</name>
    <age>99</age>
    <sportsperson>false</sportsperson>
</person>
EOF;

$person = $serializer->deserialize($data, Person::class, 'xml');
```

## Encoders

Encoders turn arrays into formats and vice versa. 
They implement EncoderInterface for encoding (array to format) and DecoderInterface for decoding (format to array).

```php
use Symfony\Component\Serializer\Encoder\JsonEncoder;
use Symfony\Component\Serializer\Encoder\XmlEncoder;
use Symfony\Component\Serializer\Serializer;

$encoders = [new XmlEncoder(), new JsonEncoder()];
$serializer = new Serializer([], $encoders);
```

### Built-in Encoders

The Serializer component provides several built-in encoders:

- JsonEncoder. This class encodes and decodes data in JSON.
- XmlEncoder. This class encodes and decodes data in XML.
- YamlEncoder. This encoder encodes and decodes data in YAML. This encoder requires the Yaml Component.
- CsvEncoder. This encoder encodes and decodes data in CSV.
