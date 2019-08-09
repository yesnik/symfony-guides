# Routes

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
