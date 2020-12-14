# Cache

In Symfony Profiler there are a number of things called *pools* - different cache systems (most are used internally by Symfony). 

That's a fancy way of saying that Symfony has two main cache systems: 

- `cache.system` (which is cleared between deploys).
- `cache.app` (which persists between deploys). The cache "pool" that's created for the "restart" signal uses `cache.app` as its parent.

The cache is stored in a `var/cache/prod` directory.

**Note:** Don’t use the Symfony reverse proxy in production. Always prefer a reverse proxy like [Varnish](https://varnish-cache.org/) on your infrastructure or a commercial CDN.

## Console commands

- `symfony console cache:clear` - clear cache. On next request new cache will be built. 
- `rm -rf var/cache/dev/http_cache/` - clear HTTP Cache

**Note:** If you need to purge the cache often, it probably means that the *caching strategy* should be tweaked 
(by lowering the TTL or by using a validation strategy instead of an expiration one).

## Example

Cache has it's own config file: `config/packages/cache.yaml`.

In `src/Controller/QuestionController.php`:

```php
// ...
use Symfony\Contracts\Cache\CacheInterface;

class QuestionController extends AbstractController
{
    /**
     * @Route("/questions/{slug}", name="app_question_show")
     * @return Response
     */
    public function show(
        $slug,
        MarkdownParserInterface $markdownParser,
        CacheInterface $cache
    )
    {
        $questionText = 'How are you *World*?';

        $questionTextParsed = $cache->get(
            'markdown_'. md5($questionText),
            function () use ($markdownParser, $questionText) {
                return $markdownParser->transformMarkdown($questionText);
            }
        );

        return $this->render('question/show.html.twig', [
            'question' => 'Question: ' . $slug,
            'questionText' => $questionTextParsed,
        ]);
    }
}
```

## Symfony HTTP Cache Kernel

To test the HTTP cache strategy, edit `public/index.php`:

```php
use Symfony\Bundle\FrameworkBundle\HttpCache\HttpCache;
// ...
$kernel = new Kernel($_SERVER['APP_ENV'], (bool) $_SERVER['APP_DEBUG']);

if ('dev' === $kernel->getEnvironment()) {
    $kernel = new HttpCache($kernel);
}
// ...
```

### Cache in controller

Let’s cache the homepage for an hour. Edit `src/Controller/ConferenceController.php`:

```php
class ConferenceController extends AbstractController
{
    // ...

    /**
     * @Route("/", name="homepage")
     */
    public function index(ConferenceRepository $conferenceRepository): Response
    {
        $response = new Response(
            $this->twig->render('conference/index.html.twig', [
                'conferences' => $conferenceRepository->findAll(),
            ])
        );
        $response->setSharedMaxAge(3600);

        return $response;
    }
```

### Check caching

```bash
curl -I http://127.0.0.1:8000/

# ...
# X-Symfony-Cache: HEAD /: fresh;
```

## Avoiding SQL Requests with ESI

When you want to cache a fragment of a page, move it outside of the current HTTP request by creating a sub-request. [ESI](https://symfony.com/doc/current/http_cache/esi.html) (Edge Side Includes) is a perfect match for this use case. An ESI is a way to embed the result of an HTTP request into another.

1. Enable ESI support. Edit `config/packages/framework.yaml`:

```yml
framework:
    esi: true
```

2. Update the Twig layout to call the controller's action. Use `render_esi` instead of `render`. Edit `templates/base.html.twig`:

```
{{ render_esi(path('conference_header')) }}
```

The cache strategy can be different from the main page and its ESIs. 
If we have an "about" page, we might want to store it for a week in the cache, and still have the header be updated every hour.
