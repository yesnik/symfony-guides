# Symfony directory structure

- `config/` -  all bundles' config files
- `config/bundles.php` - list of enabled bundles
- `public/index.php` - front controller
- `src/` - source code
- `var/cache/` - should only be used to store long term cached contents like compiled container files, compiled translations, or Doctrine proxies. No temporary files
- `var/tmp/` - use this for temporary files
- `vendor/` - folder for installed composer packages
