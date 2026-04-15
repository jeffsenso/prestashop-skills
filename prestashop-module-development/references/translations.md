# Translations & multilingual support

## Translation domains

PrestaShop uses dot-separated translation domains. The module domain follows the pattern `Modules.{ModuleName}.{Context}`:

```php
// In PHP (controller, module class, service)
$this->trans('Text to translate', [], 'Modules.Mymodule.Admin');
$this->trans('Front text', [], 'Modules.Mymodule.Shop');
```

```twig
{# In Twig templates #}
{{ 'Text to translate'|trans({}, 'Modules.Mymodule.Admin') }}
{{ 'Save'|trans({}, 'Admin.Actions') }}
```

```smarty
{* In Smarty templates *}
{l s='Text to translate' d='Modules.Mymodule.Shop'}
```

## Declaring the new translation system

Add to your main module class:

```php
public function isUsingNewTranslationSystem(): bool
{
    return true;
}
```

## Translation files

Place `.xlf` catalogue files under `translations/`:

```
translations/
├── default/
│   └── Modules.Mymodule.Admin.en.xlf
│   └── Modules.Mymodule.Admin.fr.xlf
│   └── Modules.Mymodule.Shop.en.xlf
```

Generate them with:

```bash
php bin/console translation:extract --locale=en --output-format=xlf
```

## Resources

- [Translation system](https://devdocs.prestashop-project.org/9/development/internationalization/translation/)
