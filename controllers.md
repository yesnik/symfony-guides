## Controllers

The main responsibility of a controller is to return an HTTP Response for the request.

### Create controller

```bash
symfony console make:controller
```

File `src/Controller/ArticleController.php`:

```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

class ArticleController extends AbstractController
{
    #[Route('/article/{slug}', name: 'articles')]
    public function view(string $slug): Response
    {
        return $this->json(['slug' => $slug]);
    }
}
```

The `{slug}` part of the route is a dynamic route parameter - placeholder. 
To get the value of the `{slug}` parameter add a controller argument with the same name: `string $slug`

### Autowiring

Controllers are also services that live in the container. But additionally they have the ability to autowire arguments into its methods.

Before Symfony executes the controller, it looks at each argument. 
For simple arguments like `$slug`, it passes us the wildcard value from the router:

```php
public function toggleArticleHeart($slug, LoggerInterface $logger) { ... }
```

But for type-hinted argument `LoggerInterface $logger` it looks at the *type-hint* and realizes that we want to get the logger object. 
The order of the arguments does not matter.

Autowiring works in 2 places:

- the controller actions
- the `__construct()` method of services

The container's job is to *instantiate* services. 
And so autowiring should really only work for `__construct()` functions! 
In fact, the only reason that it also works for controller actions is for convenience!

This is a very powerful idea called *autowiring*: 
if you need a service object, you just need to know the correct type-hint to use!

Every service is stored inside another object called the *container*. 
And each service has an internal name, just like routes.

And what exactly puts these services into the container? The answer: *bundles* - Symfony's plugin system.
Look at the file `config/bundles.php`. Each bundle gives a service to our app.

**What can we autowire in controller**

- type-hint a service class or interface - Symfony will give us that service;
- type-hint an entity class - Symfony will query for that entity by using the wildcard in the route;
- type-hint the `Request` class - Symfony will give you Request.

Example file `src/Controller/CommentAdminController.php`:

```php
class CommentAdminController extends AbstractController
{
    /**
     * @Route("/admin/comment", name="comment_admin")
     */
    public function index(CommentRepository $repository, Request $request)
    {
        $q = $request->query->get('q');
        $comments = $repository->findAllWithSearch($q);
        
        return $this->render('comment_admin/index.html.twig', [
            'comments' => $comments,
        ]);
    }
}
```

**RequestStack service**

While the `Request` object is not in the service container, there is a service called `RequestStack`. 
We can fetch it like any service and call `getCurrentRequest()` to get the `Request`:

```php
public function index(RequestStack $requestStack)
{
    $request = $requestStack->getCurrentRequest();
}
```

**Show list of type-hints** / **Find service**

```bash
# All services
php bin/console debug:autowiring

# Services that matches "log"
php bin/console debug:autowiring log
```

This command will show a full list of all of the type-hints that you can use to get a service. 
Notice that most of them say that they are an alias to something.
Whenever you install a new package, you'll get more and more services in this list.

### Param converter

Symfony will see the type-hint and automatically query for a `Question` object `WHERE slug =` the `{slug}` route wildcard value.

```php
    /**
     * @Route("/questions/{slug}", name="app_question_show")
     * @param Question $question
     * @return Response
     */
    public function show(Question $question)
    {
        return $this->render('question/show.html.twig', [
            'question' => $question,
        ]);
    }
```

This works because our wildcard is called `slug`, which matches the property name. 
Quite literally this makes a query where slug equals the `{slug}` part of the URL. 
If we add `{id}` in the URL, then the query would be `WHERE slug = {slug} AND id = {id}`.

It even handles the 404 for us! If we add foo to the slug in the URL... we still get a 404!

### Return JSON

```php
$this->json(['amount' => 100]);
```

### Allow only POST

```php
class PostController extends AbstractController
{
    /**
     * @Route("/posts/{id}/vote/{direction}", methods={"POST"})
     */
    public function vote($id, $direction) {}
```

### Redirect

```php
class SecurityController extends AbstractController
{
    /**
     * @Route("/register", name="app_register")
     */
    public function register(Request $request, UserPasswordEncoderInterface $passwordEncoder)
    {
        if ($request->isMethod('POST')) {
            // ...
            return $this->redirectToRoute('app_account');
        }

        return $this->render('security/register.html.twig');
    }
}
```
