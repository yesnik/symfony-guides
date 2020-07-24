# Twig

Installation:

```bash
composer req twig
```

## About twig

### Simple example

Controller file `src/Controller/QuestionController.php`:

```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class QuestionController extends AbstractController
{
    /**
     * @Route("/questions/{slug}")
     * @return Response
     */
    public function show($slug)
    {
        return $this->render('question/show.html.twig', [
            'question' => 'Question: ' . $slug,
        ]);
    }
}
```
Template file `templates/question/show.html.twig`:

```twig
<h1>{{ question }}</h1>
```

### Create url for route in twig

File: `templates/companies/index.html.twig`:

```html
<!-- It will create relative href: "/companies/1/edit" -->
<a href="{{ path('company_edit', {'id': company.id}) }}">
    Edit
</a>

<!-- It will create absolute href: "http://127.0.0.1:8000/companies/2/edit" -->
<a href="{{ url('company_edit', {'id': company.id}) }}">
    Edit
</a>
```


## Twig extensions

### Install

Twig extensions [documentation](http://twig-extensions.readthedocs.io/en/latest/).

```bash
composer require twig/extensions
```

### Create twig extension

```
bin/console make:twig-extension
```

This command will create `src/Twig/AppExtension.php`.
After this we can use created filters in our twig templates.

### Show twig functions, filters, globals

```
php bin/console debug:twig
```

### Twig variables

It twig we have `app` variable that is related to `AppVariable` class.

```
<input type="text" name="q" value="{{ app.request.query.get('q') }}">
```

### Twig Extensions: Always Instantiated

If you go to a page that renders any Twig template, then the `AppExtension` will always be instantiated, 
even if we don't use any of its custom functions or filters:

```php
class AppExtension extends AbstractExtension
{
    public function __construct(MarkdownService $markdownService)
    {
        $this->markdownService = $markdownService;
    }
    
    // ...
```

In this example we are instantiating an extra object - `MarkdownService` - on every request that uses Twig... 
even if we never actually use it! It sounds subtle, but as your Twig extension grows, 
this can become a real problem.

We can prevent this premature initialization with `ContainerInterface`:

See `src/Twig/AppExtension.php`:

```php
use Symfony\Component\DependencyInjection\ServiceSubscriberInterface;

class AppExtension extends AbstractExtension implements ServiceSubscriberInterface
{
    protected $container;

    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
    }

    public function getFilters(): array
    {
        return [
            // If your filter generates SAFE HTML, you should add a third
            // parameter: ['is_safe' => ['html']]
            // Reference: https://twig.symfony.com/doc/2.x/advanced.html#automatic-escaping
            new TwigFilter('cached_markdown', [$this, 'processMarkdown'], ['is_safe' => ['html']]),
        ];
    }

    public function processMarkdown($value): string
    {
        // Service is not instantiated until and unless we fetch it out of this container
        return $this->container
            ->get(MarkdownService::class)
            ->parse($value);
    }

    public static function getSubscribedServices()
    {
        return [
            MarkdownService::class,
        ];
    }
}
```

It this example we tell Symfony to pass us the `MarkdownService`, 
but not actually instantiate it until, and unless, we need it. 
