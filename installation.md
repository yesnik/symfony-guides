# Installation

See [docs](https://symfony.com/doc/current/setup.html)

### Way 1. Using Symfony CLI

- Install [Symfony CLI](https://symfony.com/download)
- Create a new project:
  ```
  symfony new mysite
  ```
- Start dev server:
  ```
  symfony serve
  symfony server:start -d
  ```

### Way 2. Using project skeleton

- Create a new project:
  ```
  composer create-project symfony/skeleton mysite
  ```
- Start dev server:
  ```
  php -S 127.0.0.1:8000 -t public
  ```
