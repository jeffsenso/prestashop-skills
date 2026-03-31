---
name: prestashop-module-development
description: "Complete PrestaShop module development workflow using modern architecture and best practices. Use when: creating new PrestaShop modules, updating legacy modules to modern code, implementing hooks and actions, setting up module configuration pages, adding front office features, handling database operations, implementing security measures, managing translations, or modernizing existing PrestaShop modules from legacy patterns to current standards."
---

# PrestaShop Module Development

## When to use

Use this skill for PrestaShop module development tasks such as:

• Creating new modules with modern architecture (Symfony-based controllers, services, entities)
• Refactoring legacy modules to use modern PrestaShop patterns
• Implementing hooks, actions, and event listeners
• Adding configuration pages (both legacy and modern approaches)
• Creating front office features and widgets
• Setting up database entities and migrations
• Implementing security measures (CSRF protection, input validation, SQL injection prevention)
• Adding multilingual support and translations
• Packaging modules for distribution
• Converting legacy code patterns to modern equivalents

## Inputs required

• PrestaShop version (target 8.x/9.x for modern development, minimum compatibility requirements)
• Module scope and functionality requirements
• Existing module path (if updating legacy code)
• Database schema requirements (if applicable)
• Front office integration needs (hooks, widgets, pages)
• Configuration requirements (settings, admin interface needs)
• Multilingual requirements and supported languages

## Procedure

### 0) Project structure and module identification

1. **For new modules**: Use PrestaShop Module Generator at https://validator.prestashop.com/generator for quick bootstrap
2. **For existing modules**: Analyze current structure and identify legacy patterns
3. **Standard modern structure**:
```
mymodule/
├── config/
│   ├── services.yml          # Service definitions
│   └── admin/
│       └── services.yml      # Admin-specific services
├── src/
│   ├── Controller/           # Symfony controllers
│   ├── Entity/              # Doctrine entities
│   ├── Form/                # Form types
│   └── Service/             # Business logic services
├── controllers/             # Legacy controllers (avoid in new code)
├── views/
│   ├── templates/
│   │   ├── admin/          # Admin templates
│   │   ├── front/          # Front office templates
│   │   └── hook/           # Hook templates
│   ├── css/
│   ├── js/
│   └── img/
├── translations/
├── upgrade/                 # Module upgrade scripts
├── vendor/                 # Composer dependencies
├── config.xml              # Cached module properties
├── logo.png               # 140x140px module icon
└── mymodule.php           # Main module class
```

### 1) Modern module class structure

**Main module file (`mymodule.php`)**:
```php
<?php
if (!defined('_PS_VERSION_')) {
    exit;
}

class MyModule extends Module
{
    public function __construct()
    {
        $this->name = 'mymodule';
        $this->tab = 'front_office_features';
        $this->version = '1.0.0';
        $this->author = 'Your Name';
        $this->need_instance = 0;
        $this->ps_versions_compliancy = ['min' => '8.0', 'max' => _PS_VERSION_];
        $this->bootstrap = true;

        parent::__construct();

        $this->displayName = $this->trans('My Module', [], 'Modules.Mymodule.Admin');
        $this->description = $this->trans('Module description', [], 'Modules.Mymodule.Admin');
    }

    public function install()
    {
        return parent::install()
            && $this->installDatabase()
            && $this->registerHook('displayHeader')
            && Configuration::updateValue('MYMODULE_SETTING', 'default_value');
    }

    public function uninstall()
    {
        return parent::uninstall()
            && $this->uninstallDatabase()
            && Configuration::deleteByName('MYMODULE_SETTING');
    }
}
```

### 2) Modern configuration pages

**Prefer Symfony-based admin controllers** over legacy `getContent()`:

```php
// src/Controller/Admin/ConfigurationController.php
use PrestaShopBundle\Controller\Admin\FrameworkBundleAdminController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;

class ConfigurationController extends FrameworkBundleAdminController
{
    public function indexAction(Request $request): Response
    {
        $form = $this->createForm(ConfigurationFormType::class);
        $form->handleRequest($request);
        
        if ($form->isSubmitted() && $form->isValid()) {
            // Handle form submission
        }
        
        return $this->render('@Modules/mymodule/views/templates/admin/configuration.html.twig', [
            'form' => $form->createView(),
        ]);
    }
}
```

**Legacy approach** (when Symfony controllers aren't suitable):
```php
public function getContent()
{
    if (Tools::isSubmit('submit' . $this->name)) {
        $this->postValidation();
        if (!count($this->context->controller->errors)) {
            $this->postProcess();
        }
    }
    return $this->renderForm();
}
```

### 3) Database operations and entities

**Modern approach using Doctrine entities**:
```php
// src/Entity/MyEntity.php
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Table(name="ps_my_entity")
 * @ORM\Entity
 */
class MyEntity
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;
    
    // Properties and methods
}
```

**Legacy approach** (avoid in new modules):
```php
class MyEntityObjectModel extends ObjectModel
{
    public $id;
    public static $definition = [
        'table' => 'my_entity',
        'primary' => 'id',
        'fields' => [
            'name' => ['type' => self::TYPE_STRING, 'validate' => 'isGenericName'],
        ],
    ];
}
```

### 4) Security implementation (mandatory)

**Always implement these security measures**:

```php
// CSRF token validation
if (!Tools::getToken(false)) {
    $this->context->controller->errors[] = $this->trans('Invalid token', [], 'Admin.Notifications.Error');
    return false;
}

// Input validation and sanitization
$input = Tools::getValue('input_field');
if (!Validate::isGenericName($input)) {
    $this->context->controller->errors[] = $this->trans('Invalid input', [], 'Admin.Notifications.Error');
    return false;
}

// SQL injection prevention
$sql = 'SELECT * FROM ' . _DB_PREFIX_ . 'my_table WHERE id = ' . (int)$id;
// Or using prepared statements:
$sql = new DbQuery();
$sql->select('*')
   ->from('my_table')
   ->where('id = ' . (int)$id);
```

**Admin access control**:
```php
if (!$this->context->employee->hasAccess($this->id, 'edit')) {
    throw new PrestaShopException('Access denied');
}
```

### 5) Hook implementation and front office integration

**Modern hook methods**:
```php
public function hookDisplayHeader($params)
{
    // Load CSS/JS only when needed
    if ($this->context->controller instanceof ProductController) {
        $this->context->controller->addCSS($this->_path . 'views/css/front.css');
        $this->context->controller->addJS($this->_path . 'views/js/front.js');
    }
}

public function hookDisplayProductAdditionalInfo($params)
{
    $this->smarty->assign([
        'product' => $params['product'],
        'module_setting' => Configuration::get('MYMODULE_SETTING'),
    ]);
    
    return $this->display(__FILE__, 'views/templates/hook/product_additional_info.tpl');
}
```

### 6) Translation and multilingual support

**Use translation domains**:
```php
$this->trans('Text to translate', [], 'Modules.Mymodule.Admin');
$this->trans('Front office text', [], 'Modules.Mymodule.Shop');
```

**In templates**:
```smarty
{l s='Text to translate' d='Modules.Mymodule.Shop'}
```

### 7) Legacy code conversion patterns

**Convert legacy configuration to modern**:

Legacy:
```php
public function getContent()
{
    $output = '';
    if (Tools::isSubmit('submit')) {
        // Process form
    }
    return $output . $this->displayForm();
}
```

Modern:
```php
// Use Symfony Form components and controllers
// Separate business logic into services
// Use proper template rendering
```

**Convert legacy database access**:

Legacy:
```php
Db::getInstance()->execute('INSERT INTO...');
```

Modern:
```php
// Use Doctrine entities and repositories
// Implement proper data validation
// Use transactions and error handling
```

### 8) Services and dependency injection

**Define services in config/services.yml**:
```yaml
services:
  mymodule.service.my_service:
    class: 'MyModule\Service\MyService'
    arguments:
      - '@doctrine.orm.entity_manager'
      - '@prestashop.core.admin.lang.repository'
```

## Verification

**Module installation and functionality**:
• Module installs without PHP errors or warnings
• Configuration saves correctly with proper validation
• Front office features display and function properly
• Database operations execute without errors
• Translations work in all configured languages

**Security validation**:
• CSRF tokens implemented for all forms
• Input validation prevents malicious data
• SQL queries use proper sanitization
• Admin access properly restricted

**Code quality**:
• Follow [PrestaShop Coding Standards](https://devdocs.prestashop-project.org/8/development/coding-standards/)
• No use of deprecated functions or patterns
• Proper error handling and logging
• Performance considerations (caching, database optimization)

## Validation
After completing any code change,  use Websenso PrestaShop Module DevTools for code validation and best practices checks without online validation. Run the following command from the module root:

```bash
vendor/websenso/prestashop-module-devtools/bin/lotr
```
- Run it **without any flags**
- If the command exits with errors, **fix all reported errors** before considering the task done
- Re-run the command after fixes to confirm it passes
- if you encounter errors that you cannot fix, report them before finalizing the task.

## Failure modes / debugging

**Module installation issues**:
• Check PHP error logs for fatal errors during installation
• Verify database permissions and SQL syntax
• Ensure proper hook registration syntax
• Check module name consistency (folder name = class name = file name)

**Configuration page problems**:
• Form tokens missing or incorrect
• Wrong permission checks
• Template paths incorrect
• Missing form validation

**Legacy conversion issues**:
• Deprecated function usage (use modern equivalents)
• ObjectModel vs Doctrine entity conflicts
• Legacy template syntax in new themes
• Hook compatibility between versions

**Performance problems**:
• Avoid loading unnecessary resources on every page
• Use proper caching mechanisms
• Optimize database queries
• Implement lazy loading for heavy operations

## Escalation

For canonical documentation and advanced patterns:
• [PrestaShop 8 Module Creation](https://devdocs.prestashop-project.org/8/modules/creation/)  
• [PrestaShop 9 Module Creation](https://devdocs.prestashop-project.org/9/modules/creation/) (latest)
• [Module Good Practices](https://devdocs.prestashop-project.org/8/modules/creation/good-practices/)
• [Coding Standards](https://devdocs.prestashop-project.org/8/development/coding-standards/)
• [Symfony Integration](https://devdocs.prestashop-project.org/8/development/architecture/)
• [Module Validator](https://validator.prestashop.com/) for code validation
• [PrestaShop Developer Slack](https://www.prestashop-project.org/slack/) for community support

**Module-specific resources**:
• [Payment Module Skeleton](https://github.com/PrestaShop/paymentexample)
• [Official Example Modules](https://github.com/PrestaShop/example-modules) - Complete collection of official PrestaShop examples
• [Sample Modules](https://devdocs.prestashop-project.org/8/modules/sample-modules/)