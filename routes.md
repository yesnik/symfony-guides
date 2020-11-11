# Routes

A route is the link between the request path and a PHP callable, a function that creates the HTTP response for that request.
These callables are called *controllers*.

Edit `config/routes.yaml`:

```
show:
    path: /articles/{slug}
    controller: App\Controller\ArticleController::show
```

Edit controller `src/Controller/ArticleController.php`:

```php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;

class ArticleController
{
    public function show($slug)
    {
        return new Response('Show ' . $slug);
    }
}
```

### Allow only POST requests

```
article_toggle_heart:
  path: /articles/{slug}/heart
  methods: [POST]
  controller: App\Controller\ArticleController::toggleArticleHeart
```

## Define routes via annotations

### Example 1

File `src/Controller/SecurityController.php`:

```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;

class SecurityController extends AbstractController
{
    /**
     * @Route("/login", name="app_login", methods={"POST"})
     */
    public function index()
    {
        return $this->render('security/index.html.twig', [
            'controller_name' => 'SecurityController',
        ]);
    }
}
```

This code creates `/login` route that accepts only POST requests.

### Example 2

File: `src/Controller/QuestionController.php`

```php
namespace App\Controller;

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class QuestionController
{
    /**
     * @Route("/questions/{slug}")
     * @return Response
     */
    public function show($slug)
    {
        return new Response('Show: ' . $slug);
    }
}
```
