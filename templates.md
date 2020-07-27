# Templates

### Generate link to action

**Without params**

```html
<a href="{{ path('homepage') }}">Home</a>
```
In controller we can difine name for the route via annotation:

```php
class PostController extends AbstractController
{
    /**
     * @Route("/", name="homepage")
     */
    public function homepage()
    {
        // ..
    }
```

Run the command to see available routes:

```bash
php bin/console debug:router
```

**With params**

```html
<a href="{{ path('post_show', { slug: 'php-book' }) }}">A PHP Book</a>
```

### Generate link to assets

```html
<link rel="stylesheet" href="{{ asset('css/styles.css') }}">
```
Later it will help you easily to place your assets to CDN: https://s2.somecdn.com/css/styles.css 

### Add script in template

We can inherit content of parent's template:

```html
{% block javascripts %}

    {{ parent() }}

    <script src="{{ asset('js/article_show.js') }}"></script>
{% endblock %}
```
