# Webpack Encore

Encore's job (via Webpack) is simple: to read and follow all of the require statements and create one final `app.js` (and `app.css`) that contains everything for our app.

### Installation 

Install [Encore](https://symfony.com/doc/current/frontend/encore/installation.html) in Symfony app:

```
composer require symfony/webpack-encore-bundle
npm install
```

### Console commands

```bash
# Add bootstrap package
npm install bootstrap --save-dev

# Compile assets during development
npm run watch

# On deploy: create producton build
npm run build
```

### Import bootstrap styles

Edit `/assets/css/app.css`:

```css
@import "~bootstrap";
```

### Enable bootstrap theme for forms

Edit `config/packages/twig.yaml`:

```yaml
twig:
    file_name_pattern: '*.twig'
    form_themes: ['bootstrap_5_layout.html.twig']
```
This file will be loaded from `vendor\symfony\twig-bridge\Resources\views\Form\bootstrap_5_layout.html.twig`.

Read more: [How to Work with Form Themes](https://symfony.com/doc/current/form/form_themes.html#applying-themes-to-all-forms)

### Config

Everything in Encore is configured with the file `webpack.config.js`:

```js
var Encore = require('@symfony/webpack-encore');

Encore
    // directory where compiled assets will be stored
    .setOutputPath('public/build/')
    // public path used by the web server to access the output path
    .setPublicPath('/build')

    .addEntry('app', './assets/js/app.js')
    
    // enables Sass/SCSS support
    .enableSassLoader()
```

They key part is `addEntry()`: this tells Encore to load the `assets/js/app.js` file and follow all of the `require()` statements. It will then package everything together and - thanks to the first `app` argument - output final `app.js` and `app.css` files into the `public/build` directory.

With Encore, think of your `assets/js/app.js` file like a standalone JavaScript application: it will require all of the dependencies it needs (e.g. jQuery or React), including any CSS:

```
import '../css/app.scss';

// Need jQuery? Install it with "yarn add jquery", then uncomment to require it.
// const $ = require('jquery');

console.log('Hello Webpack Encore! Edit me in assets/js/app.js');
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
