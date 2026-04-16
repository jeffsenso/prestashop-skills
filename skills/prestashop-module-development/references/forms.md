# Symfony form types — universal rules

These rules apply to **every** Symfony form in a PrestaShop module: configuration forms, CRUD entity forms, filter forms, inline forms.

---

## Boolean / toggle fields — always use `SwitchType`, NEVER `RadioType`

For any boolean/on-off field, **always use `SwitchType`** (PS UI Kit component). Never use `RadioType` or `ChoiceType` with radio buttons.

```php
use PrestaShopBundle\Form\Admin\Type\SwitchType;

$builder->add('active', SwitchType::class, [
    'label'    => $this->trans('Active', 'Admin.Global'),
    'required' => false,
]);
```

- `SwitchType` renders a proper PS toggle switch (Bootstrap + PS UI Kit) with `Yes` / `No` labels.
- The Yes option appears **second** in the underlying `<input type="radio">` pair — `SwitchType` handles this automatically.
- Works correctly with `form_theme '@PrestaShop/Admin/TwigTemplateForm/prestashop_ui_kit.html.twig'`.
- Submitted value is `'1'` (true) or `'0'` (false) as a string — cast with `(bool)` when saving.

**Why not `RadioType`?**  `RadioType` / `ChoiceType expanded=true` renders plain HTML radio inputs with no PS styling. They appear broken/unstyled in the admin UI Kit and confuse users.

**Applies to:** configuration forms, CRUD create/edit forms, bulk action forms, any in-modal form.

---

## Form theme

Always set the PS UI Kit form theme so all widgets render with correct PS Bootstrap styling:

```php
// Option A — in FormType::configureOptions (preferred)
$resolver->setDefaults([
    'form_theme' => '@PrestaShop/Admin/TwigTemplateForm/prestashop_ui_kit.html.twig',
]);

// Option B — in Twig template
{% form_theme myForm '@PrestaShop/Admin/TwigTemplateForm/prestashop_ui_kit.html.twig' %}
```

Both approaches achieve the same result; Option A keeps the theme with the form type class.

---

## Common PS form types quick reference

| Field type       | PS type class                               | Notes |
|------------------|----------------------------------------------|-------|
| Boolean toggle   | `SwitchType`                                 | Always use this; never RadioType |
| Short text       | `TextType` (Symfony)                         | |
| Long text        | `TextareaType` (Symfony)                     | |
| Integer          | `IntegerType` (Symfony) or `NumberType`      | |
| Color picker     | `ColorPickerType`                            | `PrestaShopBundle\Form\Admin\Type` |
| Country select   | `CountryChoiceType`                          | `PrestaShopBundle\Form\Admin\Type` |
| Category tree    | `CategoryChoiceTreeType`                     | `PrestaShopBundle\Form\Admin\Type` |
| Image upload     | `FileType` (Symfony)                         | validate MIME + size at boundary |
| Money            | `MoneyType` (Symfony)                        | |
| Date/time        | `DatePickerType`                             | `PrestaShopBundle\Form\Admin\Type` |

All PS-specific types are in `PrestaShopBundle\Form\Admin\Type\`.
