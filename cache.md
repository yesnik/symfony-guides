# Cache

In Symfony Profiler there are a number of things called *pools* - different cache systems (most are used internally by Symfony). The one we're using is called `cache.app`. 

In `src/Controller/ArticleController.php`:

```php
use Symfony\Component\Cache\Adapter\AdapterInterface;

// ...
public function show(AdapterInterface $cache) {
    $articleContent = 'Some content';

    $item = $cache->getItem('markdown_'.md5($articleContent));

    if (!$item->isHit()) {
        $item->set($markdown->transform($articleContent));
        $cache->save($item);
    }

    $articleContent = $item->get();
}
```
