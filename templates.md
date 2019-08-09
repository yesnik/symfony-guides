# Templates

### Generate link to action

```html
<a href="{{ path('index') }}">Home</a>
```

### Generate link to assets

```html
<link rel="stylesheet" href="{{ asset('css/styles.css') }}">
```

### Add script in template

We can inherit content of parent's template:

```html
{% block javascripts %}

    {{ parent() }}

    <script src="{{ asset('js/article_show.js') }}"></script>
{% endblock %}
```
