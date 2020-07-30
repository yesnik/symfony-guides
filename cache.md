# Cache

In Symfony Profiler there are a number of things called *pools* - different cache systems (most are used internally by Symfony). The one we're using is called `cache.app`. 

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
