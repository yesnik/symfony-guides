# Symfony directory structure

- `bin/` - contains the main CLI entry point: `console`. 
- `config/` - is made of a set of default and sensible configuration files. One file per package.
    - `config/bundles.php` - list of enabled bundles
- `public/` - is the web root directory, `index.php` is the *main entry point* for all dynamic HTTP resources.
- `src/` - hosts all the code you will write. By default, all classes under this directory use the `App` PHP namespace.
- `var/` - contains caches, logs, and files generated at runtime by the application. It is the *only directory* that needs to be writable in production.
    - `var/cache/` - should only be used to store long term cached contents like compiled container files, compiled translations, or Doctrine proxies. No temporary files
    - `var/tmp/` - use this for temporary files
- `vendor/` - folder for installed composer packages
