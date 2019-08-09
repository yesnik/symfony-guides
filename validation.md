## Symfony validation

Install [validator component](https://github.com/symfony/validator):

```
composer require validator
```

Symfony's validation is interesting because you don't apply the validation rules to the form. 
You apply them to your class via *annotations*.

See [supported constraints](https://symfony.com/doc/current/validation.html#supported-constraints).

Suppose that we have `src/Entity/Article.php`:

```php
use Symfony\Component\Validator\Constraints as Assert;

/**
 * @ORM\Entity(repositoryClass="App\Repository\ArticleRepository")
 */
class Article
{
    // Annotation here ...
    private $some_field;
}
```

**Length**

```php
/**
 * @ORM\Column(type="string", length=255)
 * @Assert\Length(
 *      min = 5,
 *      max = 50,
 *      minMessage = "Title must be at least {{ limit }} characters long",
 *      maxMessage = "Title cannot be longer than {{ limit }} characters"
 * )
 */
private $title;
```

If you try to submit the form with big title you'll get the custom error message: *Title cannot be longer than 50 characters*.

**Email**

```php
/**
 * @ORM\Column(type="string", length=180, unique=true)
 * @Assert\Email()
 */
private $email;
```

**NotBlank**

```php
/**
 * @ORM\Column(type="string", length=180, unique=true)
 * @Assert\NotBlank(message="Please enter an email")
 */
private $email;
```

**The Callback Constraint**

This is the tool when you need to do something totally custom. Create a method in your class and add `@Assert\Callback()` above it. Then, during validation, Symfony will call your method. 

```php
/**
 * @Assert\Callback
 */
public function validate(ExecutionContextInterface $context, $payload)
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

**UniqueEntity**

We apply unique validation to Entity:

```php
/**
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 * @UniqueEntity(
 *     fields={"email"},
 *     message="I think you're already registered!"
 * )
 */
class User implements UserInterface
{
```

### Add validation to FormType

We usually add validation rules via annotations on a class. 
But, if you have a field that's not mapped, you can add its validation rules directly to the form field via a `constraints` array option. 
What do you put inside? Remember how each annotation is represented by a concrete class? 
That's the key! Instantiate those as objects here: `new NotBlank()`. 
To pass options, use an array and set message to *Choose a password!*.

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
