# Translations & multilingual support

## ⚠️ CRITICAL: PrestaShop custom trans() signatures

PrestaShop uses **different parameter order** than standard Symfony translation for `trans()` in form types and controllers.

### In TranslatorAwareType (Form Types)

**Signature:** `trans($key, $domain, $parameters = [])`

```php
// ✅ CORRECT - domain is 2nd parameter
$this->trans('Carousel speed', 'Modules.Mymodule.Admin')
$this->trans('Label with %var%', 'Modules.Mymodule.Admin', ['%var%' => $value])

// ❌ WRONG - will throw "Argument #2 ($parameters) must be of type array, string given"
$this->trans('Carousel speed', [], 'Modules.Mymodule.Admin')
```

**Location:** `PrestaShopBundle\Form\Admin\Type\TranslatorAwareType`

### In FrameworkBundleAdminController (Controllers)

**Signature:** `trans($key, $domain, array $parameters = [])`

```php
// ✅ CORRECT
$this->trans('Successful update.', 'Admin.Notifications.Success')
$this->trans('Item %s saved', 'Modules.Mymodule.Admin', ['%s' => $name])

// ❌ WRONG - will throw "Argument #3 ($parameters) must be of type array, string given"  
$this->trans('Successful update.', [], 'Admin.Notifications.Success')
```

**Location:** `PrestaShopBundle\Controller\Admin\FrameworkBundleAdminController`

### Why this exists

PrestaShop wraps Symfony's `TranslatorInterface` with custom implementations to:
- Support PrestaShop's legacy translation system alongside Symfony's
- Handle module-specific translation domains automatically
- Maintain backward compatibility with older modules

### Standard Symfony (for reference)

Standard Symfony signature (NOT used in PrestaShop forms/controllers):
```php
trans($id, array $parameters = [], $domain = null, $locale = null)
```

### Common mistakes

1. **Using empty array for no parameters:** Always omit the 2nd argument if no parameters needed
2. **Copy-pasting from Symfony docs:** PrestaShop's signature is different
3. **Validation constraint messages in FormType:** Use plain strings, NOT trans() calls:

```php
// ✅ CORRECT - constraint messages as plain strings
new GreaterThanOrEqual([
    'value' => 1000,
    'message' => 'Speed must be at least 1000 milliseconds',
])

// ❌ WRONG - Symfony handles constraint translation separately
new GreaterThanOrEqual([
    'value' => 1000,
    'message' => $this->trans('Speed must be at least...', 'Modules.Mymodule.Admin'),
])
```

---

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
