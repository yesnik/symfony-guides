# Authentication & Authorization

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

Authorization is all about denying access to read or perform different operations... and this is enforced in a way that's independent of how you log in.

There are 2 main ways to handle authorization:

- access_control in `security.yaml`
- denying access in your controller

**1. access_control in security.yaml**

When a user logs in - no matter how they authenticate or where your user data is stored - your login mechanism assigns that user a *set of roles*. In our app, those roles are stored in the database and we'll eventually let admin users modify them via our API. The simplest way to prevent access to an endpoint is by making sure *the user has some role*. 

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

`access_control` is great for some situations, but most of the time you'll need *more flexibility*. 
In a traditional Symfony app, we add security to controllers. But in API Platform we don't have any controllers! Ok, so instead of thinking about protecting each controller, we'll think about protecting *each API operation*. Maybe we want this collection GET operation to be accessible anonymously but we want to require a user to be authenticated in order to POST and create a new CheeseListing.

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

### Authentication in javascript

If you're consuming your API from JavaScript, you have 2 basic options for authentication:

1. You can use `HttpOnly` cookies, which is how sessions work.
2. You can return *access token* on login and store it in JavaScript. 

Storing access tokens in JavaScript is a dangerous practice because it can be stolen 
if some bad JavaScript somehow runs on your page. 
If the access token have a short lifetime, that helps... but then they're less useful, 
because your user will need to constantly log in.

That's why we recommend to use `HttpOnly` cookie-based authentication (like a session) for JavaScript frontend. 
It can also be used in other situations, like for authenticating a mobile app. 

But if you use `HttpOnly` cookies, then you need to worry about CSRF protection... unless you use SameSite cookies... which protects *almost every* browser... or you could use CSRF tokens to be safest... but it complicates your app.

**Avoiding CSRF Attacks**

*CSRF token* - an extra field that must be sent on form submit that proves that the request originated from the real site - not from somewhere else. Symfony's form component adds `CSRF` tokens automatically.

But using CSRF tokens in an API is annoying: you need to manage CSRF tokens and send that field manually from your JavaScript on every request. If you're using cookie-based authentication and need to 100% prevent a CSRF attack for an endpoint, this is the time-tested way to do that.

**SameSite Cookies**

There is a new way to prevent CSRF attacks - a solution that is implemented inside browsers themselves. It's called a "SameSite" cookie.

The basic reason that CSRF attacks are possible is that when a user submits the form that lives on the "bad" site, any cookies that our domain set are sent with that request to our app... even though the request isn't "originating" from our domain. For most cookies that... should probably not happen. Instead, we should be able to say:

*Hey browsers! See this session cookie that my Symfony app is setting? I want you to only send that back to my app if the request originates from my domain.*

Symfony uses `SameSite` attribute already. File `config/packages/framework.yaml`:

```yaml
framework:
    session:
        cookie_samesite: lax
```

## Login / logout routes

Login, logout routes for cookie-based authentication.

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

## Voters

If you're just checking for a role, no problem: use `is_granted('ROLE_ADMIN')`. 
But if your logic gets any more complex, use a voter.

### Create voter

```bash
php bin/console make:voter
```
This command will create a file `src/Security/Voter/CheeseListingVoter.php`.

### How voters work

Whenever you call `is_granted()`, Symfony loops through all of the "voters" in the system and asks each one: *Current user has EDIT access to this `Post` object?*

Symfony has 2 core voters: 

1. The first knows how to decide access when you call `is_granted()` and pass it `ROLE_` something, like `ROLE_USER` or `ROLE_ADMIN`. It determines that by looking at the roles on the authenticated user. 
2. The second voter knows how to decide access if you call `is_granted()` and pass it one of the `IS_AUTHENTICATED_` strings: `IS_AUTHENTICATED_FULLY`, `IS_AUTHENTICATED_REMEMBERED` or `IS_AUTHENTICATED_ANONYMOUSLY`.

Now that we've created a class and made it extend Symfony's Voter base class, our app has a third voter. This means that, whenever someone calls `is_granted()`, Symfony will call the `supports()` method and pass it the `$attribute` - that's the string `EDIT`, or `ROLE_USER` - and the `$subject`, which will be the `CheeseListing` object in our case.

Our job here is to answer the question: do we know how to decide access for this `$attribute` and `$subject` combination? Or should another voter handle this?

We're going to design our voter to decide access if the `$attribute` is `EDIT` and if `$subject` is an `instanceof CheeseListing`.

If anything else is passed (e.g. `ROLE_ADMIN`) `supports()` will return `false` and Symfony will know to ask a different voter.

But if we return `true` from `supports()`, Symfony will call `voteOnAttribute()` and pass us the same `$attribute` string - `EDIT` - the same `$subject` - `CheeseListing` object - and a `$token`, which contains the authenticated `User` object. Our job in this method is clear: return `true` if the user should have access or `false` if they should not.

```php
// src/Security/Voter/CheeseListingVoter.php

namespace App\Security\Voter;

use App\Entity\CheeseListing;
use Symfony\Component\Security\Core\Authentication\Token\TokenInterface;
use Symfony\Component\Security\Core\Authorization\Voter\Voter;
use Symfony\Component\Security\Core\User\UserInterface;

class CheeseListingVoter extends Voter
{
    protected function supports($attribute, $subject)
    {
        // replace with your own logic
        // https://symfony.com/doc/current/security/voters.html
        return in_array($attribute, ['EDIT'])
            && $subject instanceof CheeseListing;
    }

    protected function voteOnAttribute($attribute, $subject, TokenInterface $token)
    {
        $user = $token->getUser();
        // if the user is anonymous, do not grant access
        if (!$user instanceof UserInterface) {
            return false;
        }
        /** @var CheeseListing $subject */
        // ... (check conditions and return true to grant permission) ...
        switch ($attribute) {
            case 'EDIT':
                if ($subject->getOwner() === $user) {
                    return true;
                }
                return false;
        }
        throw new \Exception(sprintf('Unhandled attribute "%s"', $attribute));
    }
}
```
