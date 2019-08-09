## Symfony forms

Install form component:

```
composer require form
```

Create form / generate form:

```
php bin/console make:form
```

Call the class, `UserRegistrationFormType`. This will ask if you want this form to be bound to a class. 
That's usually what we want, but it's optional. Bind our form to the `User` class.


### Method 1. With using data_class

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

Edit view `templates/article_admin/new.html.twig`:

**Short version**

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

**Descriptive version**

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

**Add bootstrap design to forms**

If you're using Bootstrap CSS or Foundation CSS, ah, you're in luck! Symfony comes with a built-in form theme that makes your forms render exactly how these systems want.

```
twig:
    form_themes:
        - bootstrap_4_layout.html.twig
```
This template lives deep inside the core of Symfony.


### Method 2. Manual way

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
    /**
     * @Route("/admin/article/new", name="admin_article_new")
     * @IsGranted("ROLE_ADMIN_ARTICLE")
     */
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
    
    /**
     * @Route("/admin/article/{id}/edit", name="admin_article_edit")
     * @IsGranted("MANAGE", subject="article")
     */
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
But also, each individual field is also represented as its own `Form` object, and it's a child of that top-level form. Yep, `$form['plainPassword']` gives us a `Form` object that knows everything about this one field. When we call `->getData()` on it, yep! That's the value for this one field.

### Form theming

When Symfony renders any part of your form, it looks for a specific block in this 
core `form_div_layout.html.twig` template. 
For example, to render the "row" part of any field, it looks for `form_row`. 
Also this system has some hierarchy to it: to render the `label` part of a TextType field, 
it first looks for `text_label` and then falls back to using `form_label`.
