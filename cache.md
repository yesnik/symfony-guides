# Cache

In Symfony Profiler there are a number of things called *pools* - different cache systems (most are used internally by Symfony). 

That's a fancy way of saying that Symfony has two main cache systems: 

- `cache.system` (which is cleared between deploys).
- `cache.app` (which persists between deploys). The cache "pool" that's created for the "restart" signal uses `cache.app` as its parent.

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

## Clear cache

```bash
php bin/console cache:clear
```

On next request new cache will be built. 
The cache is stored in a `var/cache/prod` directory.

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
