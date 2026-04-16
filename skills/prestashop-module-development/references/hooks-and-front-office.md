# Hooks & front office integration

## Registering hooks

Register hooks via the `Installer`:

```php
private array $hooks = ['displayHeader', 'displayFooter', 'displayProductAdditionalInfo'];
```

## Hook method patterns

```php
// Load CSS/JS only when needed
public function hookDisplayHeader($params)
{
    if ($this->context->controller instanceof ProductController) {
        $this->context->controller->addCSS($this->_path . 'views/css/front.css');
        $this->context->controller->addJS($this->_path . 'views/js/front.js');
    }
}

// Render a Smarty template from a hook
public function hookDisplayProductAdditionalInfo($params)
{
    $this->smarty->assign([
        'product' => $params['product'],
        'module_setting' => Configuration::get('MYMODULE_SETTING'),
    ]);

    return $this->display(__FILE__, 'views/templates/hook/product_additional_info.tpl');
}
```

## Widget interface (front office data provider)

Implement `WidgetInterface` when the module exposes a widget to themes:

```php
use PrestaShop\PrestaShop\Core\Module\WidgetInterface;

class MyModule extends Module implements WidgetInterface
{
    public function renderWidget($hookName = null, array $configuration = []): string
    {
        if (!$this->isCached($this->templateFile, $this->getCacheId($hookName))) {
            $this->smarty->assign($this->getWidgetVariables($hookName, $configuration));
        }
        return $this->fetch($this->templateFile, $this->getCacheId($hookName));
    }

    public function getWidgetVariables($hookName = null, array $configuration = []): array
    {
        return [
            'items' => $this->getSomeData(),
        ];
    }
}
```

## Back-office header hook (injecting JS vars)

Use `hookDisplayBackOfficeHeader` to inject JavaScript variables and assets only for the relevant controller:

```php
public function hookDisplayBackOfficeHeader()
{
    $controller = Tools::getValue('controller');
    if ($controller === 'AdminMymoduleMyentity') {
        Media::addJsDef([
            'myVar' => 'value',
        ]);
        $this->context->controller->addJS($this->_path . 'views/js/admin.js');
    }
}
```

## Resources

- [Hooks reference](https://devdocs.prestashop-project.org/9/modules/concepts/hooks/)
- [Widget interface](https://devdocs.prestashop-project.org/9/modules/concepts/widget/)
