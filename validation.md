## Symfony validation

Install [validator component](https://github.com/symfony/validator):

```
composer require validator
```

Symfony's validation is interesting because you don't apply the validation rules to the form. 
You apply them to your class via PHP attributes or annotations (PHP < 8.0).

See [supported constraints](https://symfony.com/doc/current/validation.html#supported-constraints).

Suppose that we have `src/Entity/Article.php`:

```php
use Symfony\Component\Validator\Constraints as Assert;

#[ORM\Entity(repositoryClass: ArticleRepository::class)]
class Article
{
    // PHP Attribute here ...
    private $someField;
}
```

**NotBlank** - [docs](https://symfony.com/doc/current/reference/constraints/NotBlank.html)

```php
#[Assert\NotBlank(message="Please enter an description")]
#[ORM\Column(length: 255)]
private ?string $description = null;
```

**Length** - [docs](https://symfony.com/doc/current/reference/constraints/Length.html)

```php
#[Assert\Length(
    min: 2,
    max: 50,
    minMessage: 'Title must be at least {{ limit }} characters long',
    maxMessage: 'Title cannot be longer than {{ limit }} characters',
)]
#[ORM\Column(length: 255)]
private $title;
```

If you try to submit the form with big title you'll get the custom error message: *Title cannot be longer than 50 characters*.

**Email** - [docs](https://symfony.com/doc/current/reference/constraints/Email.html)

```php
#[Assert\Email(
    message: 'The email {{ value }} is not a valid email.',
)]
#[ORM\Column(type="string", length=180, unique=true)]
private ?string $email = null;
```

**The Callback Constraint**

This is the tool when you need to do something totally custom. 
Create a method in your class and add `Assert\Callback()` above it. Then, during validation, Symfony will call your method. 

```php
#[Assert\Callback]
public function validate(ExecutionContextInterface $context, mixed $payload): void
{
    if (stripos($this->getTitle(), 'bad word') !== false) {
        $context->buildViolation('Do not use bad word for title!')
            ->atPath('title')
            ->addViolation();
    }
}
```

*Important:* Because the callback lives inside your entity, you *don't have access to any services*. 
So, you couldn't make a query, for example. If Callback doesn't work, the solution that always works is to create your very *own custom validation constraint*.

**UniqueEntity** - [docs](https://symfony.com/doc/current/reference/constraints/UniqueEntity.html)

This validation won't allow us to create 2 users with the same `username` OR 2 users with the same `email`:

```php
// DON'T forget the following use statement!!!
use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;

#[ORM\Entity(repositoryClass: UserRepository::class)]
#[UniqueEntity('username')]
#[UniqueEntity('email')]
class User implements UserInterface
{
    #[ORM\Column(name: 'email', type: 'string', length: 255, unique: true)]
    #[Assert\Email]
    protected string $email;
}
```

### Add validation to FormType

We usually add validation rules via annotations on a class. 
But, if you have a field that's not mapped, you can add its validation rules directly to the form field via a `constraints` array option. 
What do you put inside? Remember how each annotation is represented by a concrete class? 
That's the key! Instantiate those as objects here: `new NotBlank()`. 

```php
use Symfony\Component\Validator\Constraints\Length;
use Symfony\Component\Validator\Constraints\NotBlank;

class UserRegistrationFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('email')
            // don't use password: avoid EVER setting that on a
            // field that might be persisted
            ->add('plainPassword', PasswordType::class, [
                'mapped' => false,
                'constraints' => [
                    new NotBlank([
                        'message' => 'Choose a password!'
                    ]),
                    new Length([
                        'min' => 5,
                        'minMessage' => 'Come on, you can think of a password longer than that!'
                    ])
                ]
            ]);
        ;
    }
    
    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => User::class,
        ]);
    }
}
```
