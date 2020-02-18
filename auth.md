# Authorization

## Create authenticator

```
php bin/console make:auth
```

Choose `Empty authenticator` and define `LoginFormAuthenticator` as the class name of authenticator.
This command will create file `src/Security/LoginFormAuthenticator.php`.

```php
use Symfony\Component\Security\Guard\Authenticator\AbstractFormLoginAuthenticator;

class LoginFormAuthenticator extends AbstractFormLoginAuthenticator
{
    public function supports(Request $request)
    {
        // do your work when we're POSTing to the login page
        return $request->attributes->get('_route') === 'app_login'
            && $request->isMethod('POST');
    }
    
    // ..
}
```

This class basically has a method for each step of the authentication process. Symfony calls its `supports()` method at the beginning of every request.

If we return `false` from `supports()`, nothing else happens. Symfony doesn't call any other methods on our authenticator, and the request continues on like normal to our controller, like nothing happened. It's not an authentication failure - it's just that nothing happens at all.

## Authorization

*Authorization* is all about deciding whether or not a user should have access to something. 
This is where, for example, you can require a user to log in before they see some page - or restrict some sections to admin users only.

There are 2 main ways to handle authorization:

- access_control in `security.yaml`
- denying access in your controller

**1. access_control in security.yaml**

Edit `config/packages/security.yaml`:

```yaml
security:
    access_control:
        - { path: ^/admin, roles: ROLE_ADMIN }
```

The path is a regular expression. So, this access control says that any URL that starts with `/admin` should require a role called `ROLE_ADMIN`.

Access controls work like routes: Symfony checks them one-by-one from top to bottom. 
And as soon as it finds one access control that matches the URL, it uses that and stops. 
So maximum of one access control is used on each page load.

**2. Use IsGranted annotation**

Edit `/src/Controller/CommentAdminController.php`:

```php
use Sensio\Bundle\FrameworkExtraBundle\Configuration\IsGranted;

class CommentAdminController extends AbstractController
{
    /**
     * @Route("/admin/comment", name="comment_admin")
     * @IsGranted("ROLE_ADMIN")
     */
    public function index(CommentRepository $repository, Request $request, PaginatorInterface $paginator)
    {
        // ...
    }
}
```

Also we can add annotation `@IsGranted("ROLE_ADMIN")` above controller to apply it to every class' method.

**Is user logged in**

There are 2 ways to check whether or not the user is simply logged in: 

1) By checking `ROLE_USER`. File `templates/base.html.twig`:
```php
{% if is_granted('ROLE_USER') %}
    Create article
{% endif %}
```

2) Edit `config/packages/security.yaml`:
```php
security:
    access_control:
        - { path: ^/account, roles: IS_AUTHENTICATED_FULLY }
```
It's just a special string that simply checks if the user is logged in or not. 

**Protect all pages except login**

Edit `config/packages/security.yaml`:

```
security:
    access_control:
        # but, definitely allow /login to be accessible anonymously
        - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        # if you wanted to force EVERY URL to be protected
        - { path: ^/, roles: IS_AUTHENTICATED_FULLY }
```

This one is weird. Who has `IS_AUTHENTICATED_ANONYMOUSLY`? Everyone! 
If you're anonymous, you have it. If you're logged in, you have it too!

**Role hierarchy**

Symfony has a feature called *role_hierarchy*. Edit: `config/packages/security.yaml`:

```
security:
    role_hierarchy:
        ROLE_ADMIN: [ROLE_ADMIN_COMMENT, ROLE_ADMIN_ARTICLE]
```

It's just that simple. Now, anybody that has ROLE_ADMIN also has these two roles, automatically.
This is even cooler than you might think! It allows us to organize our roles into different groups of people in our company. 
For example, `ROLE_EDITOR` could be given access to all the sections that "editors" need. 
Then, the only role that you need to assign to an editor user is this one role: `ROLE_EDITOR`. 
And if all editors need access to a new section in the future, just add that new role to role_hierarchy.

**Impersonation, switch user**

Edit `config/packages/security.yaml`:

```
security:
    firewalls:
        main:
            switch_user: true
```

As soon as you do this, you can go to any URL and add `?_switch_user=` and the email address of a user that you want to impersonate. Let's try `spacebar1@example.com`.

To prevent any user from taking advantage of this little trick, 
the `switch_user` feature requires you to have a special role called `ROLE_ALLOWED_TO_SWITCH`. Edit `security.yaml`:

```
security:
    role_hierarchy:
        ROLE_ADMIN: [ROLE_ADMIN_COMMENT, ROLE_ADMIN_ARTICLE, ROLE_ALLOWED_TO_SWITCH]
```

To exit and return to your normal identity just add `?_switch_user=_exit` to any URL. And... we're back to being us!

To add a Banner when you are Impersonating, edit `templates/base.html.twig`:

```
{% if is_granted('ROLE_PREVIOUS_ADMIN') %}
    <div class="alert alert-warning" style="margin-bottom: 0;">
        You are currently switched to this user.
        <a href="{{ path('app_homepage', {'_switch_user': '_exit'}) }}">Exit Impersonation</a>
    </div>
{% endif %}
```

## Login / logout routes

File `config/packages/security.yaml`:

```php
security:
    firewalls:
        main:
            json_login:
                # Name of a route to login
                check_path: app_login
                username_path: email
                password_path: password
            logout:
                # Name of a route to log out
                path: app_logout
```

File `src/Controller/SecurityController.php`:

```php
namespace App\Controller;

use ApiPlatform\Core\Api\IriConverterInterface;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Annotation\Route;
use Symfony\Component\HttpFoundation\Response;

class SecurityController extends AbstractController
{
    /**
     * @Route("/login", name="app_login", methods={"POST"})
     */
    public function login(IriConverterInterface $iriConverter)
    {
        if (!$this->isGranted('IS_AUTHENTICATED_FULLY')) {
            return $this->json([
                'error' => 'Invalid login request: check that the Content-Type header is "application/json".'
            ], 400);
        }

        return new Response(null, 204, [
            'Location' => $iriConverter->getIriFromItem($this->getUser())
        ]);
    }

    /**
     * @Route("/logout", name="app_logout")
     */
    public function logout()
    {
        throw new \Exception('should not be reached');
    }
}
```
