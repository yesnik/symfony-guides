# Translation

To translate website install component:

```bash
composer require translation
```

## Translation steps

1. Edit template `templates/base.html.twig`:

```
{{ 'Conference Guestbook'|trans }}
```
2. Create / update translation file:

```bash
symfony console translation:update ru --force --domain=messages
```

This command will create `translations/messages+intl-icu.ru.xlf`.

3. Edit this file:

```
<trans-unit id="eOy4.6V" resname="Conference Guestbook">
  <source>Conference Guestbook</source>
  <target>Гостевая книга конференций</target>
</trans-unit>
```
