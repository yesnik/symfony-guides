# Data Transformers

See [docs](https://symfony.com/doc/current/form/data_transformers.html)

Data transformers are used to translate the data for a field into a format that can be displayed in a form. 

They're already used internally for many field types. For example, the `DateType`: 

- Data transformer converts the `DateTime` value of the field to a `yyyy-MM-dd` formatted string when rendering the form.
- Data transformer converts string back to a `DateTime` object on submit.

## Example

Suppose we have a form with a text field "Tags". We want to enter tags there and separate them by comma. 

### TagTransformer

```php
namespace App\Form\DataTransformer;

use App\Entity\Tag;
use App\Repository\TagRepository;
use Doctrine\Common\Collections\ArrayCollection;
use Symfony\Component\Form\DataTransformerInterface;
use Symfony\Component\Form\Exception\TransformationFailedException;

final readonly class TagTransformer implements DataTransformerInterface
{
    public function __construct(
        private TagRepository $tagRepository,
    ) {
    }

    /**
     * Transforms an object (Array Collection) to a string (tags).
     *
     * @param ArrayCollection<int, Tag>|null $value
     */
    public function transform($value): string
    {
        if ($value === null) {
            return '';
        }

        $tagNames = [];
        foreach ($value as $tag) {
            $tagNames[] = $tag->getName();
        }

        return implode(', ', $tagNames);
    }

    /**
     * Transforms a string (tag names) to an object (ArrayCollection).
     *
     * @param  string $value
     * @throws TransformationFailedException if object (tag) is not found.
     */
    public function reverseTransform($value): ArrayCollection
    {
        if (!$value) {
            return new ArrayCollection();
        }

        $tagNames = explode(',', $value);
        $tagNames = array_map('trim', $tagNames);
        $tagNames = array_unique($tagNames);

        $tags = new ArrayCollection();

        foreach ($tagNames as $tagName) {
            $tag = $this->tagRepository->findOneBy(['name' => $tagName]);
            if (!$tag) {
                $tag = (new Tag())->setName($tagName);
            }

            $tags->add($tag);
        }

        return $tags;
    }
}
```

### BlogType

File `src\Form\BlogType.php`:

```php
class BlogType extends AbstractType
{
    public function __construct(
        private readonly TagTransformer $tagTransformer,
    ) {}

    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder
            ->add('title')
            // ...
            ->add('tags', TextType::class, [
                'required' => false,
            ])
        ;

        $builder->get('tags')
            ->addModelTransformer($this->tagTransformer);
    }
    // ...
}
```

### Entity Blog

File: `src\Entity\Blog.php`:

```php
#[ORM\Entity(repositoryClass: BlogRepository::class)]
class Blog
{
    // ...
    /**
     * @var Collection<int, Tag>
     */
    #[ORM\ManyToMany(targetEntity: Tag::class, inversedBy: 'blogs', cascade: ['persist'])]
    private Collection $tags;

    public function __construct()
    {
        $this->tags = new ArrayCollection();
    }
    // ...
}
```
