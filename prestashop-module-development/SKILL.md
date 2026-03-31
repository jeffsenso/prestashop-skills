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
• Adding configuration pages (modern approaches)
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
3. **Namespace naming convention** — derive the PSR-4 namespace from the module name:
   - The module name prefix (first 2 letters before the real words) becomes the **top-level vendor namespace**
   - The remaining real words become **sub-namespaces** in CamelCase (detect word boundaries)
   - **Never** use the generic `PrestaShop\Module\` vendor prefix — it is reserved for PrestaShop core modules
   - Example: `wsproductpaymentlogos` → prefix `Ws`, words `Product`, `Payment`, `Logos` → `Ws\ProductPaymentLogos`
   - Example: `mycompanycoolfeature` → prefix `My` (2 letters), words `Company`, `Cool`, `Feature` → `My\CompanyCoolFeature`
   - This applies to `composer.json` `autoload.psr-4`, PHP `namespace` declarations, and `config/services.yml` FQCNs
4. **Standard modern structure**:
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

> **Rule**: Never put `Configuration::updateValue/deleteByName`, hook registration, or DB queries directly in `install()`/`uninstall()`. Delegate entirely to an `Installer` class from `src/Install/`.

> **Critical**: Always `require_once __DIR__ . '/vendor/autoload.php';` after the `_PS_VERSION_` guard in the main module file. Without it, the module's own namespaced classes (`Installer`, form types, controllers…) will not be found when PrestaShop loads the module (e.g. `php bin/console pr:mo install …` will throw _"Attempted to load class … Did you forget a use statement?"_).

```php
<?php
if (!defined('_PS_VERSION_')) {
    exit;
}

require_once __DIR__ . '/vendor/autoload.php';

use Vendor\MyModule\Install\Installer;
use PrestaShop\PrestaShop\Adapter\SymfonyContainer;

class MyModule extends Module
{
    public function __construct()
    {
        $this->name = 'mymodule';
        $this->tab = 'front_office_features';
        $this->version = '1.0.0';
        $this->author = 'Your Name';
        $this->need_instance = 0;
        $this->ps_versions_compliancy = ['min' => '1.7.8.0', 'max' => _PS_VERSION_];
        $this->bootstrap = true;

        parent::__construct();

        $this->displayName = $this->trans('My Module', [], 'Modules.Mymodule.Admin');
        $this->description = $this->trans('Module description', [], 'Modules.Mymodule.Admin');
    }

    public function install(): bool
    {
        if (!parent::install()) {
            return false;
        }
        $installer = new Installer();
        return $installer->install($this);
    }

    public function uninstall(): bool
    {
        $installer = new Installer();
        return $installer->uninstall() && parent::uninstall();
    }

    // For Symfony-routed configuration tab:
    public function getTabs(): array
    {
        return [[
            'class_name' => 'AdminMymoduleConfiguration',
            'visible' => false,
            'name' => 'My Module Configuration',
            'parent_class_name' => 'CONFIGURE',
            'route_name' => 'mymodule_configuration',
        ]];
    }

    public function getContent(): void
    {
        $router = SymfonyContainer::getInstance()->get('router');
        Tools::redirectAdmin($router->generate('mymodule_configuration'));
    }
}
```

### 1a) Installer pattern (`src/Install/`)

**Always split install logic into dedicated classes** in `src/Install/`:

```
src/Install/
├── Installer.php              # orchestrates install/uninstall
├── ConfigurationInstaller.php # handles Configuration table values
├── FixturesInstaller.php      # (optional) inserts default/sample data
└── index.php                  # PS security guard
```

**`src/Install/Installer.php`** — orchestrator:
```php
namespace Vendor\MyModule\Install;

class Installer
{
    private array $hooks = ['displayHeader', 'displayFooter'];

    private ConfigurationInstaller $configurationInstaller;
    // private FixturesInstaller $fixturesInstaller; // only if default data needed

    public function __construct()
    {
        $this->configurationInstaller = new ConfigurationInstaller();
        // $this->fixturesInstaller = new FixturesInstaller();
    }

    public function install(\Module $module): bool
    {
        if (!$this->registerHooks($module)) {
            return false;
        }
        // if (!$this->installDatabase()) { return false; } // only if DB tables needed
        if (!$this->configurationInstaller->install()) {
            return false;
        }
        // $this->fixturesInstaller->install(); // only if default data needed
        return true;
    }

    public function uninstall(): bool
    {
        // $this->uninstallDatabase(); // only if DB tables exist
        return $this->configurationInstaller->uninstall();
    }

    private function registerHooks(\Module $module): bool
    {
        return (bool) $module->registerHook($this->hooks);
    }

    // Only add installDatabase/uninstallDatabase + executeQueries if module has DB tables:
    // private function installDatabase(): bool { return $this->executeQueries(SqlQueries::installQueries()); }
    // private function uninstallDatabase(): bool { return $this->executeQueries(SqlQueries::uninstallQueries()); }
    // private function executeQueries(array $queries): bool {
    //     foreach ($queries as $query) {
    //         if (!\Db::getInstance()->execute($query)) { return false; }
    //     }
    //     return true;
    // }
}
```

**`src/Install/ConfigurationInstaller.php`** — installs config per shop context:
```php
namespace Vendor\MyModule\Install;

use Configuration;
use Shop;

class ConfigurationInstaller
{
    public function install(): bool
    {
        $shops = Shop::getContextListShopID();
        $shopGroups = [];
        $res = true;

        foreach ($shops as $shopId) {
            $groupId = (int) Shop::getGroupFromShop($shopId, true);
            if (!in_array($groupId, $shopGroups)) {
                $shopGroups[] = $groupId;
            }
            $res &= (bool) Configuration::updateValue('MYMODULE_SETTING', 'default', false, $groupId, $shopId);
        }
        foreach ($shopGroups as $groupId) {
            $res &= (bool) Configuration::updateValue('MYMODULE_SETTING', 'default', false, $groupId);
        }
        $res &= (bool) Configuration::updateValue('MYMODULE_SETTING', 'default');

        return (bool) $res;
    }

    public function uninstall(): bool
    {
        return (bool) Configuration::deleteByName('MYMODULE_SETTING');
    }
}
```

**`src/Install/FixturesInstaller.php`** — only create when module needs default data:
```php
namespace Vendor\MyModule\Install;

class FixturesInstaller
{
    public function install(): void
    {
        // Insert default entities, sample content, etc.
    }
}
```

### 2) Modern configuration pages

> **⚠️ DO NOT use `HelperForm`** — it is based on Smarty + Bootstrap 3 and is explicitly discouraged in Symfony controllers. See https://devdocs.prestashop-project.org/9/development/components/helpers/helperform/
> **⚠️ DO NOT use `getContent()` to render HTML** — only use it to redirect to the Symfony route.

The canonical pattern uses four classes and two config files:

```
config/routes.yml                           # declares the Symfony route
config/services.yml                         # wires all services + controller
src/Form/ConfigurationDataConfiguration.php # reads/writes PS configuration table
src/Form/ConfigurationFormDataProvider.php  # bridges form ↔ DataConfiguration
src/Form/ConfigurationFormType.php          # Symfony form type (no HelperForm!)
src/Controller/Admin/ConfigurationController.php  # handles GET/POST, renders Twig
views/templates/admin/configuration.html.twig     # Twig template with PS UI Kit
```

**`config/routes.yml`** — link route to controller and legacy tab:
```yaml
mymodule_configuration:
  path: /mymodule/configuration
  methods: [GET, POST]
  defaults:
    _controller: 'Vendor\MyModule\Controller\Admin\ConfigurationController::index'
    _legacy_controller: AdminMymoduleConfiguration
    _legacy_link: AdminMymoduleConfiguration
```

> **Namespace rule**: derive from module name — `mymodule` = `My\Module`, `xscoolfeature` = `Xs\CoolFeature`.
> Never use the generic `PrestaShop\Module\` vendor prefix.

**`composer.json` autoload**:
```json
"autoload": {
  "psr-4": { "Vendor\\MyModule\\": "src/" }
}
```

**`config/services.yml`** — wire all four classes:
```yaml
services:
  _defaults:
    public: true

  prestashop.module.mymodule.form.configuration_data_configuration:
    class: Vendor\MyModule\Form\ConfigurationDataConfiguration
    arguments:
      - '@prestashop.adapter.legacy.configuration'

  prestashop.module.mymodule.form.configuration_data_provider:
    class: Vendor\MyModule\Form\ConfigurationFormDataProvider
    arguments:
      - '@prestashop.module.mymodule.form.configuration_data_configuration'

  prestashop.module.mymodule.form.type.configuration:
    class: Vendor\MyModule\Form\ConfigurationFormType
    parent: 'form.type.translatable.aware'
    tags:
      - { name: form.type }

  prestashop.module.mymodule.form.configuration_data_handler:
    class: PrestaShop\PrestaShop\Core\Form\Handler
    arguments:
      - '@form.factory'
      - '@prestashop.core.hook.dispatcher'
      - '@prestashop.module.mymodule.form.configuration_data_provider'
      - 'Vendor\MyModule\Form\ConfigurationFormType'
      - 'MymoduleConfiguration'

  Vendor\MyModule\Controller\Admin\ConfigurationController:
    public: true
    arguments:
      - '@prestashop.module.mymodule.form.configuration_data_handler'
      - '@prestashop.adapter.legacy.configuration'
    tags:
      - { name: controller.service_arguments }
```

**`src/Form/ConfigurationDataConfiguration.php`** — reads/writes PS config table:
```php
final class ConfigurationDataConfiguration implements DataConfigurationInterface
{
    public const CONFIG_TITLE = 'MYMODULE_TITLE';

    public function __construct(private ConfigurationInterface $configuration) {}

    public function getConfiguration(): array
    {
        return ['title' => $this->configuration->get(self::CONFIG_TITLE) ?? ''];
    }

    public function updateConfiguration(array $configuration): array
    {
        $this->configuration->set(self::CONFIG_TITLE, $configuration['title'] ?? '');
        return [];
    }

    public function validateConfiguration(array $configuration): bool { return true; }
}
```

**`src/Form/ConfigurationFormType.php`** — Symfony form using PS UI Kit types:
```php
class ConfigurationFormType extends TranslatorAwareType
{
    public function buildForm(FormBuilderInterface $builder, array $options): void
    {
        $builder->add('title', TextType::class, [
            'label' => $this->trans('Title', 'Modules.Mymodule.Admin'),
            'required' => false,
        ]);
    }

    public function configureOptions(OptionsResolver $resolver): void
    {
        parent::configureOptions($resolver);
        $resolver->setDefaults([
            'form_theme' => '@PrestaShop/Admin/TwigTemplateForm/prestashop_ui_kit.html.twig',
        ]);
    }
}
```

**`src/Controller/Admin/ConfigurationController.php`** — handles the request:
```php
class ConfigurationController extends FrameworkBundleAdminController
{
    public function __construct(
        private FormHandlerInterface $formHandler,
        private ConfigurationInterface $configuration
    ) {}

    public function index(Request $request): Response
    {
        $form = $this->formHandler->getForm();
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $errors = $this->formHandler->save($form->getData());
            if (empty($errors)) {
                $this->addFlash('success', $this->trans('Successful update.', 'Admin.Notifications.Success'));
                return $this->redirectToRoute('mymodule_configuration');
            }
            foreach ($errors as $error) { $this->addFlash('danger', $error); }
        }

        return $this->render(
            '@Modules/mymodule/views/templates/admin/configuration.html.twig',
            ['configurationForm' => $form->createView()]
        );
    }
}
```

**`views/templates/admin/configuration.html.twig`** — Twig template:
```twig
{% form_theme configurationForm '@PrestaShop/Admin/TwigTemplateForm/prestashop_ui_kit.html.twig' %}
{% extends '@PrestaShop/Admin/layout.html.twig' %}
{% block content %}
  {{ form_start(configurationForm) }}
  <div class="card">
    <h3 class="card-header"><i class="material-icons">settings</i> {{ 'Settings'|trans({}, 'Admin.Global') }}</h3>
    <div class="card-body"><div class="form-wrapper">{{ form_widget(configurationForm) }}</div></div>
    <div class="card-footer"><div class="d-flex justify-content-end">
      <button type="submit" class="btn btn-primary">{{ 'Save'|trans({}, 'Admin.Actions') }}</button>
    </div></div>
  </div>
  {{ form_end(configurationForm) }}
{% endblock %}
```

**`mymodule.php`** — register tab, redirect `getContent()` to Symfony route:
```php
public function getTabs(): array
{
    return [[
        'class_name' => 'AdminMymoduleConfiguration',
        'visible' => false,
        'name' => 'My Module Configuration',
        'parent_class_name' => 'CONFIGURE',
        'route_name' => 'mymodule_configuration',
    ]];
}

public function getContent(): void
{
    $router = SymfonyContainer::getInstance()->get('router');
    Tools::redirectAdmin($router->generate('mymodule_configuration'));
}
```

Reference implementation: https://github.com/PrestaShop/example-modules/tree/master/demosymfonyform

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

> **AI agent rule — NEVER SKIP EITHER STEP**: Both steps below are **mandatory** after every code change session, without exception.
> - Do NOT stop after Step 1 (lotr). Step 2 (install test) is equally required.
> - Do NOT ask the user whether to run them — just run them, every time.
> - Do NOT consider the task complete until both steps have been executed and passed.
> - Skipping or deferring either step is a validation failure, even if the code looks correct.

### Step 1 — Dev tools (run from the module root)

```bash
vendor/websenso/prestashop-module-devtools/bin/lotr
```

- Run it **without any flags** — it applies all fixers (autoindex, header stamp, PS version checker, PS validator, PHPStan, PHP-CS-Fixer) in sequence.
- **Working directory must be the module root** (`/path/to/modules/mymodule`), not the PS root or a subdirectory.
- Expected output: `🎉 All commands completed successfully!` with `Executed: 6/6 commands successfully`.
- If any step fails:
  1. Read the full error output carefully.
  2. Fix all reported errors in the relevant files.
  3. Re-run `lotr` until all 6 steps pass.
  4. If an error cannot be fixed (e.g. PHPStan baseline needed), document it and report to the user before finalising.
- `lotr` may auto-modify files (index.php guards, file headers, CS fixes) — those changes are correct and should be committed.

### Step 2 — Module installation test (run from the PS root)

```bash
cd /path/to/prestashop && php bin/console pr:mo install mymodule
```

- Replace `mymodule` with the actual module folder name (e.g. `wsproductpaymentlogos`).
- **Working directory must be the PS root**, not the module directory — the console command is at `bin/console`.
- Expected output (French PS): `L'action Install sur le module … a réussi.`
- Expected output (English PS): `Action Install on module … succeeded.`
- If the install fails:
  - Check for `Attempted to load class` → missing `require_once __DIR__ . '/vendor/autoload.php';` in the main module file.
  - Check for `Call to undefined method` → wrong namespace or missing `use` statement.
  - Check for DB errors → review `ConfigurationInstaller` and any SQL queries in `Installer`.
  - Re-run after fixes.
- If the module was already installed, : `php bin/console pr:mo uninstall mymodule`, then re-install.
- After a successful install, clear the cache if config routes are not resolving: `php bin/console cache:clear`.

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
• [demosymfonyform](https://github.com/PrestaShop/example-modules/tree/master/demosymfonyform) - **canonical Symfony form configuration page example**
• [HelperForm (discouraged)](https://devdocs.prestashop-project.org/9/development/components/helpers/helperform/) - legacy only, do not use in new code
• [Sample Modules](https://devdocs.prestashop-project.org/8/modules/sample-modules/)