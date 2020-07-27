# Templates

### Generate link to action

```html
<a href="{{ path('index') }}">Home</a>
```
Run command to see available routes:

```bash
php bin/console debug:router
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
