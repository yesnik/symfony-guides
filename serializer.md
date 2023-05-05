# Symfony Serializer Component

[Docs](https://symfony.com/doc/current/components/serializer.html)

 An *array* is used as an intermediary between objects and serialized contents. 
 This way, encoders will only deal with turning specific formats into arrays and vice versa. 
 The same way, *Normalizers* will deal with turning specific objects into arrays and vice versa.
 
 ```bash
 composer require symfony/serializer
 ```
 
 ## Serializing an Object
 
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
