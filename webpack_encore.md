# Webpack Encore

### Installation 

Install [Encore](https://symfony.com/doc/current/frontend/encore/installation.html) in Symfony app:

```
composer require encore
yarn install
```

### Config

Everything in Encore is configured via a `webpack.config.js` file at the root of your project. 

With Encore, think of your `assets/js/app.js` file like a standalone JavaScript application: it will require all of the dependencies it needs (e.g. jQuery or React), including any CSS:

```
require('../css/app.css');

// Need jQuery? Install it with "yarn add jquery", then uncomment to require it.
// const $ = require('jquery');

console.log('Hello Webpack Encore! Edit me in assets/js/app.js');
```

### Compile assets

Encore's job (via Webpack) is simple: to read and follow all of the require statements and create one final `app.js` (and `app.css`) that contains everything your app needs.

Compile assets during development:

```
yarn encore dev --watch
```

### Include assets in template

Edit base template `templates/base.html.twig`:

```twig
{% block stylesheets %}
    {# 'app' must match the first argument to addEntry() in webpack.config.js #}
    {{ encore_entry_link_tags('app') }}

    <!-- Renders a link tag (if your module requires any CSS)
         <link rel="stylesheet" href="/build/app.css"> -->
{% endblock %}

{% block javascripts %}
    {{ encore_entry_script_tags('app') }}

    <!-- Renders app.js & a webpack runtime.js file
        <script src="/build/runtime.js"></script>
        <script src="/build/app.js"></script> -->
{% endblock %}
```
