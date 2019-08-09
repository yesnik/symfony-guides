## Flash message

Use `$this->addFlash()` in your controller, edit `src/Controller/ArticleAdminController.php`:

```php
class ArticleAdminController extends AbstractController
{
    /**
     * @Route("/admin/article/new", name="admin_article_new")
     */
    public function new(EntityManagerInterface $em, Request $request)
    {
        $form = $this->createForm(ArticleFormType::class);

        $form->handleRequest($request);
        if ($form->isSubmitted() && $form->isValid()) {
            // ...
            $this->addFlash('success', 'Article Created! Knowledge is power!');

            return $this->redirectToRoute('app_homepage');
        }
```

Display flash in the template `templates/base.html.twig`:

```
{% for message in app.flashes('success') %}
    <div class="alert alert-success">
        {{ message }}
    </div>
{% endfor %}
```
