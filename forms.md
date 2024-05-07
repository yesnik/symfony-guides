# Symfony forms

Install [form](https://github.com/symfony/form) component:

```bash
composer require form
```

Create form / generate form:

```bash
symfony console make:form
```

Call the class, `UserRegistrationFormType`. This will ask if we want this form to be bound to a class. 
That's usually what we want, but it's optional. Bind our form to the `User` class.

## Form Builder

### Options

```php
$builder->add('title', TextType::class, [
    'required' => true,
    'help' => 'Some title for your Blog',
    'attr' => [
        'class' => 'title-header',
    ]
]);
```

#### required

```php
$builder->add('title', TextType::class, [
    'required' => true,
]);
```

It adds `required` attribute to the form:

```html
<input type="text" id="blog_title" name="blog[title]" required="required" class="form-control">
```

#### help

```php
$builder->add('title', TextType::class, [
    'help' => 'Some title for your Blog',
]);
```

It adds help text below the field:

```html
<div id="blog_title_help" class="form-text mb-0 help-text">Some title for your Blog</div>
```

#### attr

```php
$builder->add('title', TextType::class, [
    'attr' => [
        'class' => 'title-header'
    ]
]);
```

It adds custom attributes to the input.

#### mapped

```php
$builder
    ->setMethod('GET')
    ->add('content', TextType::class, [
        'required' => false,
        'mapped' => false,
    ])
```

Data of the content field won't be mapped to the Entity. We can get value in the controller:

```php
$blog = new Blog();
$form = $this->createForm(BlogType::class, $blog);
$form->handleRequest($request);

if ($form->isSubmitted() && $form->isValid()) {
    dd($form->get('content')->getData());
}
```

### EntityType

```php
$builder->add('category', EntityType::class, [
    'class' => Category::class,
    'placeholder' => 'No category',
    'required' => false,
    'query_builder' => function (CategoryRepository $er): QueryBuilder {
        return $er->createQueryBuilder('u')
            ->orderBy('u.name', 'ASC');
    },
    'choice_label' => 'name',
]);
```

## Method 1. Create form using `data_class`

Create `src/Form/ArticleFormType.php`:

```php
namespace App\Form;

use App\Entity\Article;
use App\Entity\User;
use Symfony\Bridge\Doctrine\Form\Type\EntityType;
use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormBuilderInterface;
use Symfony\Component\OptionsResolver\OptionsResolver;

class ArticleFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('title', TextType::class, [
                'help' => 'Choose something catchy!'
            ])
            ->add('content')
            ->add('author', EntityType::class, [
                'class' => User::class,
                'choice_label' => function(User $user) {
                    return sprintf('(%d) %s', $user->getId(), $user->getEmail());
                },
                'placeholder' => 'Choose an author'
            ])
            ->add('publishedAt', null, [
                'widget' => 'single_text'
            ])
        ;
        ;
    }

    public function configureOptions(OptionsResolver $resolver)
    {
        $resolver->setDefaults([
            'data_class' => Article::class
        ]);
    }
}
```

Edit controller `src/Controller/ArticleAdminController.php`:

```php
class ArticleAdminController extends AbstractController
{
    /**
     * @Route("/admin/article/new", name="admin_article_new")
     * @IsGranted("ROLE_ADMIN_ARTICLE")
     */
    public function new(EntityManagerInterface $em, Request $request)
    {
        $form = $this->createForm(ArticleFormType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            /** @var Article $article */
            $article = $form->getData();
            $article->setAuthor($this->getUser());

            $em->persist($article);
            $em->flush();

            $this->addFlash('success', 'Article Created! Knowledge is power!');

            return $this->redirectToRoute('app_homepage');
        }

        return $this->render('article_admin/new.html.twig', [
            'articleForm' => $form->createView()
        ]);
    }
```

You should never instantiate the form type directly. Instead, use the `createForm()` method.
This method is part of `AbstractController` and eases the creation of forms.

When passing a form to a template, use `createView()` to convert the data to a format suitable for templates.

Edit view `templates/article_admin/new.html.twig`:

### Short version

```php
{% extends 'content_base.html.twig' %}

{% block content_body %}
    <h1>Launch a new Article!</h1>

    {{ form_start(articleForm) }}

        {{ form_widget(articleForm) }}
        
        <button type="submit" class="btn btn-primary">Create!</button>
    {{ form_end(articleForm) }}
{% endblock %}
```

### Descriptive version

Instead of `form_widget` we can use `form_row` to describe each form's field:

```php
{% extends 'content_base.html.twig' %}

{% block content_body %}
    <h1>Launch a new Article!</h1>

    {{ form_start(articleForm) }}

        {# Displaying of one from_row is splitted into 4 methods; #}
        {{ form_label(articleForm.title, 'Article title') }}
        {{ form_errors(articleForm.title) }}
        {{ form_widget(articleForm.title) }}
        {{ form_help(articleForm.title) }}

        {{ form_row(articleForm.author, {label: 'Autor of the article'}) }}
        {{ form_row(articleForm.content) }}
        {{ form_row(articleForm.publishedAt) }}

        <button type="submit" class="btn btn-primary">Create!</button>
    {{ form_end(articleForm) }}
{% endblock %}

```

### Add bootstrap design to forms

Symfony comes with a built-in form theme that makes your forms render exactly how Bootstrap CSS or Foundation CSS want.

```
twig:
    file_name_pattern: '*.twig'
    form_themes: ['bootstrap_5_layout.html.twig']
```

## Method 2. Create form manually

If you have a super complex form that looks different than your entity, it's perfectly okay to not use `data_class`. 
Sometimes it's simpler to build the form exactly how you want, call `$form->getData()` and use that associative array in your controller to update what you need.

Create `src/Form/ArticleFormType.php`:

```php
namespace App\Form;

use Symfony\Component\Form\AbstractType;
use Symfony\Component\Form\FormBuilderInterface;

class ArticleFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('title')
            ->add('content')
        ;
    }
}
```

Edit controller `src/Controller/ArticleAdminController.php`:

```php
class ArticleAdminController extends AbstractController
{
    #[Route("/admin/article/new", name="admin_article_new")]
    #[IsGranted("ROLE_ADMIN_ARTICLE")]
    public function new(EntityManagerInterface $em, Request $request)
    {
        $form = $this->createForm(ArticleFormType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            $data = $form->getData();

            $article = new Article();
            $article->setTitle($data['title']);
            $article->setContent($data['content']);
            $article->setAuthor($this->getUser());

            $em->persist($article);
            $em->flush();

            $this->addFlash('success', 'Article Created! Knowledge is power!');

            return $this->redirectToRoute('app_homepage');
        }

        return $this->render('article_admin/new.html.twig', [
            'articleForm' => $form->createView()
        ]);
    }
    
    #[Route('/admin/article/{id}/edit', name='admin_article_edit')]
    #[IsGranted('MANAGE', subject='article')]
    public function edit(Article $article, Request $request, EntityManagerInterface $em)
    {
        $form = $this->createForm(ArticleFormType::class, $article);
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $em->persist($article);
            $em->flush();

            $this->addFlash('success', 'Article Updated! Inaccuracies squashed!');

            return $this->redirectToRoute('admin_article_edit', [
                'id' => $article->getId(),
            ]);
        }
        return $this->render('article_admin/edit.html.twig', [
            'articleForm' => $form->createView()
        ]);
    }
```

### Add field to form without mapping to entity

When you add fileds to form use `['mapped' => false]` option.

Edit `src/Form/UserRegistrationFormType.php`:

```php
class UserRegistrationFormType extends AbstractType
{
    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
            ->add('email')
            // don't use password: avoid EVER setting that on a
            // field that might be persisted
            ->add('plainPassword', PasswordType::class, ['mapped' => false])
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

You can read this variable at controller via `$form['plainPassword']->getData()`:

```php
class SecurityController extends AbstractController
{
    /**
     * @Route("/register", name="app_register")
     */
    public function register(Request $request)
    {
        $form = $this->createForm(UserRegistrationFormType::class);

        $form->handleRequest($request);

        dd($form['plainPassword']->getData()); // "pass123"
```

*Explanation:* When you call `$this->createForm()`, it creates a `Form` object that represents the whole form. 
But also, each individual field is also represented as its own `Form` object, and it's a child of that top-level form. 
Yep, `$form['plainPassword']` gives us a `Form` object that knows everything about this one field. 
When we call `->getData()` on it we get the value for this one field.

## Form theming

When Symfony renders any part of your form, it looks for a specific block in this 
core `form_div_layout.html.twig` template. 
For example, to render the "row" part of any field, it looks for `form_row`. 
Also this system has some hierarchy to it: to render the `label` part of a TextType field, 
it first looks for `text_label` and then falls back to using `form_label`.

## CSRF protection disable

By default Symfony generate hidden field inside form tag:

```html
<input type="hidden" id="blog__token" name="blog[_token]" value="21f74a218fd687e9780.sgIYNSskXtC6wKT4NVph0yyzagLBm4IUXOioHBGbKBw.611RYl9XCeXps82xByMWhnbBJjK17-kjKLKef3TyeErYekxnWGcHmtDw0w">
```

We can disable it:

```php
class BlogType extends AbstractType
{
    public function configureOptions(OptionsResolver $resolver): void
    {
        $resolver->setDefaults([
            'data_class' => Blog::class,
            'csrf_protection' => false,
        ]);
    }
}
```
