## Controllers

### Create controller

```
bin/console make:controller
```

File `src/Controller/ArticleController.php`:

```php
namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

class ArticleController extends AbstractController
{
    public function toggleArticleHeart($slug)
    {
        return $this->json(['hearts' => rand(5, 100)]);
    }
}
```

### Autowiring

Before Symfony executes our controller, it looks at each argument. For simple arguments like $slug, it passes us the wildcard value from the router:

```php
public function toggleArticleHeart($slug, LoggerInterface $logger) { ... }
```

But for `$logger` (we call it *type-hinted argument*), it looks at the *type-hint* and realizes that we want Symfony to pass us the logger object. 
Oh, and the order of the arguments does not matter.

Autowiring works in 2 places:

- the controller actions
- the `__construct()` method of services

The container's job is really to *instantiate* services. 
And so autowiring should really only work for `__construct()` functions! 
In fact, the only reason that it also works for controller actions is for convenience!

This is a very powerful idea called autowiring: 
if you need a service object, you just need to know the correct type-hint to use!

Every service is stored inside another object called the *container*. 
And each service has an internal name, just like routes.

And what exactly puts these services into the container? The answer: *bundles* - Symfony's plugin system.

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

```
./bin/console debug:autowiring
```

This command will show a full list of all of the type-hints that you can use to get a service. 
Notice that most of them say that they are an alias to something.
Whenever you install a new package, you'll get more and more services in this list.

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
