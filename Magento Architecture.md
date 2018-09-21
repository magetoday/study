#Magento Architecture


##DESCRIBE MAGENTO’S MODULE-BASED ARCHITECTURE

	Modules live in one of two places:
		• app/code/CompanyName/ModuleName
		• vendor/vendor-name/module-name

	With Magento 2, modules can be installed with Composer, making for easier upgrades and version management. These modules are placed in /vendor. Any file in /vendor is considered “untouchable” (not directly editable) as those files 	will be removed when new versions are released. Installing modules via Composer into the /vendor folder reduces technical debt as you no longer have to directly manage the versions of these modules: Composer does it for you.
	In most cases, the modules that you develop go into app/code/CompanyName. 
	This is the namespace used for development. Modules are placed in the folder directly below. With Magento’s dependency on Composer, you can theoretically import modules from almost any location in a project.

	The only required files to initiate a module is:
		• registration.php
		• etc/module.xml

	registration.php	
		This file is included by the Composer autoloader (app/etc/NonComposerComponentRegistration.php). 
		This adds the module (component) to the static list of components in  \Magento\Framework\Component\ComponentRegistrar. Once it is there, Magento will look for etc/module.xml

	etc/module.xml
		This file specifies the setup version and loading sequence for the module. The setup version is available in the module’s Setup classes to determine what upgrade activities should occur.
		The sequence tells Magento (vendor/magento/framework/Module/ModuleList/Loader.php) in which order to load modules. This adjustment happens only when the module is installed. The list of modules is stored in app/etc/config.php
		The sequence is useful for events, plugins and preferences, (although events’ listeners should never be dependent on the order in which they are called) and layouts. If you modify the layout of another module and your module is initialized first, the other module will overwrite your adjustments.

	Describe module limitations.
		Magento 2 modules are to be completely contained within the module’s directory (ie. app/code/AcmeWidgets/ProductPromoter). All customizations and extensions are made within that folder.
		If it is disabled, the module is not functional. Keep in mind that disabling a module does not automatically remove its tables or entries in the database.
		If you are having trouble with modules not being enabled, check:
			• The required files are present (see above).
			• The module is enabled (bin/magento module:enable AcmeWidgets_ProductPromotor).


	How do different modules interact with each other?
		Magento 2 provides a much-improved system for inter-module interaction. At its core, Magento 2 is PSR-4 compliant. This standard declares that the namespace path will match the file path to the class.
		While the PSR-0 standard has been deprecated, Composer still uses this idea to assist in autoloading classes. In /composer.json (in the root folder), there should be a “psr-0” node. Inside it, will be an “app/code” value. This has the same effect of adding an include directory for PHP to search.
		Magento 2 does not include the Mage “god” class like its earlier counterpart. 
		Magento 2 relies on service contracts and dependency injection to determine what classes to use. This will be discussed more in the Dependency Injection section.
		These service contracts are stored in the app/code/AcmeWidgets/ProductPromoter/Api/ directory. Data representations are in the app/code/AcmeWidgets/ProductPromoter/Api/Data folder. 
		When building modules that are dependent on other modules, whenever possible, use interfaces as defined by the service contracts. That way, your module works with a blueprint and it does not matter what class or module fulfills that blueprint.
	
	What side effects can come from this interaction?
		Any time you are working with a system where any installed module can extend core functionality, fallout can happen.
		When implementing according to Magento 2’s guidelines, the fallout is limited. When breaking recommendations, problems happen.
		It is good to assume that other modules will change things in the system. For example, let’s say you install a module that introduces a massive change for a product. This change is so significant that the developer wrote an entirely new class to represent the product model. This class was written against the service contracts that Magento specifies for a product:

		But what if your custom product code uses a method that is only defined in \Magento\Catalog\Model\Product? You (or your customer or a website visitor) might get a PHP Fatal Error when it attempts to call a method that does not exist in the other class. This flexibility is incredible but also must be used responsibly.
		When working with Magento-defined classes that have service contracts (Api/ and Api/Data folders), use those interfaces instead of the concrete classes that implement the service contracts. If you must explicitly use a Magento-defined class (such as \Magento\Catalog\Model\Product), make the implicit dependency explicit by depending on the concrete implementation instead of the service contract interface.

##DESCRIBE MAGENTO’S DIRECTORY STRUCTURE
	
	Determine how to locate different types of files in Magento.
	Magento 2 brings a very flexible system for building and structuring modules. The below answers will be a hybrid of Magento’s practices and the freedom that you now have with naming files and folders.
	Magento 2’s core files are found in /vendor/magento or /app/code/Magento(only to be used for Magento core code contributions). Some supporting JS and CSS files are found in /lib.

	/Api: Service Contracts
		The /Api folder stores the contracts for modules that contain specific actions that can be reliably utilized from various places in the app. An example of this would be \Magento\Catalog\Api\CategoryListInterface.

	/Api/Data: Data Service Contacts
		This folder contains interfaces that represent data. Examples of this would be a Product interface, a Category interface or a Customer interface. The concrete implementations of these interfaces usually do little more than provide getters and setters for data (a.k.a Data Transfer Objects).

	/Block: View Models (or template “assistants”)
		Very little PHP should be done in templates (separation of responsibilities: see hereand here). As a result, Magento’s blocks perform or provide business logic for the templates. In the MVVM (model, view, view-model) methodology Magento utilizes, Blocks are “View Models.”
		One problem with inheriting the template block is the constructor is very large. This makes testing and reusability difficult. The answer to this is using a View Model. While View Models live in the Block/ directory, they are very different. A view Model provides the business logic of a block, and not the rendering functionality. This makes another distinction in code where the block is responsible for outputting its contents and the view model handles the logic.

	/Console: Console Commands
		When running bin/magento on the command line, a list of available commands to run is output. The code for commands should reside in the / Console folder. An example of a console command is for the bin/magento catalog:product-attributes:cleanup command: /vendor/magento/module-catalog/Console/Command/ProductAttributesCleanUp.php.

	/Controller: Web Request Handlers
		When a page is requested from a Magento website, the path is parsed out and matched to parameters found in routes.xml and files found in the /Controller folder.

	/Controller/Adminhtml: Admin controllers
		Remember back in the Magento 1 “good-old-days” where all of the Magento adminhtml files were in app/code/core/Mage/Adminhtml (Mage_Adminhtml module)? Thankfully, those days are over with Magento 2. Admin controllers now are spread through their respective and applicable modules.

	/Cron: Cron Job Classes
		This folder is the standard place for storing actions that are executed on a schedule (cron job). 

	/etc: Configuration files
		Configuration files are placed here. Examples include: module.xml, crontab. xml, config.xml, webapi.xml, di.xml.
		Any files directly within the /etc folder are global. The area of those files can be controlled by putting it within a folder with the name of the target area. For example, a di.xml within /etc/frontend would only apply to the front end.
		Some files have to be within an area folder (e.g. routes.xml and sections. xml), some have to be global (e.g. acl.xml) and some can be global or area specific (e.g. di.xml).

	/Helper: Occasionally useful for small, reusable code
		With Magento 2, this folder is no longer necessary. However, the core still utilizes this folder for storing actions. In my experience, here are good rules to apply for functionality:
		Every method should be capable of being declared static (whether or not it is). No transient data (other than injected classes) is stored in a helper class.

	/i18n: Translation CSV Files
		This is where all a module’s translation CSV files are located. In Magento 1, these were in app/locale/de_DE/. There are CSV files with two columns: From, To

	/Model: Data Handling and Structures
		In the MVVM acronym, this directory houses the Models. I have found that many classes that are placed in the /Helper folder out to be in /Model. This folder stores anything that is related to the data structure (vendor/magento/module-catalog/Model/Product.php) to loading that data (vendor/magento/module-catalog/Model/ProductRepository.php).

	/Model/ResourceModel: Database Interactions
		These represent how data is retrieved and saved in the database. Any direct database interaction should happen in these files. Here is an example of the product’s Resource model (vendor/magento/module-catalog/Model/ResourceModel/Product.php).

	/Observer: Event Listeners
		When Magento fires an event, listeners that are attached to it are called. Thisdecouples the system. Magento Commerce integrates with RabbitMQ which allowseven more control and reliability to this process. Event data should be able to bestored and then run in a queue at a later time.
		Event listeners must implement the ObserverInterface (\Magento\Framework\Event\ObserverInterface). The observer class should be understandable as the intent of its function. The PHP class should follow the stand of using TitleCase while the event name should be snake_case. Avoid putting business logic into an observer. Put the logic in another location and inject that class into your Observer. When you need to use the logic elsewhere in the system, you will thank yourself.

	/Plugin: Function Modification
		Plugins are one of the most powerful features in Magento 2. They allow you to modify or change the functionality for almost any class or interface. Plugins only work on objects that are instantiated by the ObjectManager.
		Plugins can be placed anywhere. Magento puts them in the /Plugins folder which makes them easy to find. I prefer to name the plugin file after the class that I am customizing followed by “Plugin.” If I am modifying a method in \Magento\Catalog\Api\Data\ProductInterface, I would create the following class: /Plugins/Magento/Catalog/ProductPlugin.php. This reduces confusion when you import classes. Another approach is to name Plugins after what they do instead of the class. This makes it easier to locate the correct one when working with the PHPStorm Magento 2 plugin (e.g. MinimumPricePlugin instead of ProductPlugin on a Plugin into Model\Product)

	/Setup: Database Modification

		• InstallSchema.php: sets up table / column schema when the module is installed.
		• UpgradeSchema.php: modifies table / column schema when the module version is upgraded.
		• Recurring.php: runs after every install or upgrade.
		• InstallData.php: sets up data when the module is installed. An example would be adding a custom CMS block.
		• UpgradeData.php: modifies data after the module is installed, when the module version is upgraded.
		• RecurringData.php: applies to data after every install or upgrade.


	/Test:
		This houses the module’s tests. /Test/Unit stores unit tests. You can run tests via the command line: bin/magento dev:tests:run [all, unit, integration, integration-all, static, static-all, integrity, legacy, default]

	/Ui: UI Component Data Providers
		This folder stores the data providers and modifiers for UI components 
		/view/[area]/layout: Layout XML DirectivesLayout
		XML links blocks (/Blocks) with templates. It is a very flexible means to control what is displayed where.

	/view/[area]/Templates: Block Templates
		The counterpart to a Block is a template to render the HTML for the block. While the block (in /Block) represents the business logic, the template represents how the results of the business logic are shown to the user. Again, the best is for as little PHP as possible to go in the templates.
		When building modules, keep in mind that this folder here is plural “templates.”

	/view/[area]/ui_component: UI Components
		The checkout on the frontend is also a UI component, but this is configured with layout XML and not UI Component XML. You can also find a /templates folder inside the ui_component folder that contains some templating for UI components
		This folder contains the XML configuration for UI Components. They are used to represent distinct UI elements, such as grids and forms, and were designed for flexible user interface rendering. Most Magento admin grids (such as the Catalog Product grid and the Customer grid) are composed with UI Components. The checkout on the frontend is also a UI component.

	view/[area]/web: Web Assets
		This is where the web assets are stored. This includes JS, CSS (or LESS / SCSS), and images.

	/view/[area]/web/template: JS Templates
		HTML templates which can be asynchronously requested with Javascript (possible using Magento’s customization of KnockoutJS) are placed here. These files often contain KnockoutJS declarative bindings. This folder is singular "template"—which can be confusing as the PHTML and the XHTML template directories are plural. 

	/view/[area]/requirejs-config.js
		The module’s RequireJS configuration. This configuration is used to control Javascript module dependencies, create aliases, and declare mixins.

	Where are the files containing JavaScript, HTML, and PHP located?
		JavaScript: in the /view/[area]/web/ folder
		HTML: in the /view/[area]/templates folder (with .phtml file extensions)
		PHP: any folder except for /view/[area]/web/

	How do you find the files responsible for certain functionality?
		The files and folder list above detail out what files are stored where.

##UTILIZE CONFIGURATION XML AND VARIABLES SCOPE
	
	Magento’s XML configuration is split across multiple files, depending on the purpose. This helps avoid having one very large configuration file.

	Additionally, Magento 2 supports configuration based on the area. This can be seen in vendor/magento/magento-catalog/etc/ where there are the following folders: adminhtml, frontend, webapi_rest, webapi_soap. The configuration that is in these folders is only loaded if Magento is initialized in that area. For example, when browsing the admin panel, the configuration found in frontend, webapi_rest or webapi_soap is not loaded.
	We will discuss the more important XML files found in the /etc folder.

		module.xml
			This is the only required configuration file. It specifies the current module’s version and the module loading order. Example: vendor/magento/module-catalog/etc/config.xml
		
		acl.xml
			This defines the permissions for accessing protected resources. Example: vendor/magento/module-catalog/etc/acl.xml

		config.xml
			This loads in default configuration in to Store > Configuration. This is also where configuration entries can be marked as encrypted (password). See vendor/magento/module-fedex/etc/config.xml

		crontab.xml
			This identifies actions that are to occur on a schedule. Example: vendor/magento/module-catalog/etc/crontab.xml

		di.xml
			This configures dependency injection for your module. This is perhaps the most frequently used file when customizing Magento. Here plugins are defined, class substitutions performed, concrete classes are specified for interfaces, virtual types setup, and constructor arguments can be specified or modified. 
			It is very important to familiarize yourself with the capabilities of this file. Example: vendor/magento/module-catalog/etc/di.xml

		email_templates.xml
			Specifies email templates that are used in Magento. The template id is the concatenated XML-style path to where in system configuration template is specified.Example: vendor/magento/module-customer/etc/email_templates.xml

		events.xml
			This file registers event listeners. This file can often be put into a specific area.Example: vendor/magento/module-catalog/etc/frontend/events.xml

		indexer.xml
			Configures Magento indexers. Example: vendor/magento/module-catalog/etc/indexer.xml

		adminhtml/menu.xml
			Configures the menu in the adminhtml area. Example: vendor/magento/module-customer/etc/adminhtml/menu.xml

		mview.xml
			Triggers a type of event when data is modified in a database column (materialized views). This is most often used for indexing.Example: vendor/magento/module-catalog/etc/mview.xml
		
		[area]/routes.xml
			Tells Magento that this area accepts web requests. The route node configures the first part of the layout handle (route ID) and the front name (first segment in the URL 	after the domain name).

		adminhtml/system.xml
			Specifies configuration tabs, sections, groups and fields found in Store Configuration. Example: vendor/magento/module-customer/etc/adminhtml/system.xml

		view.xml
			Similar to config.xml but used for specifying default values for design configuration.Example: vendor/magento/theme-frontend-luma/etc/view.xml

		webapi.xml
			Configures API access and routes. Example: vendor/magento/module-catalog/etc/webapi.xml

		widget.xml
			Configures widgets to be used in products, CMS pages, and CMS blocks.Example: vendor/magento/module-catalog/etc/widget.xml


	Determine how to use configuration files in Magento

		Magento’s XML configuration is loaded on an as-needed basis. When di.xml is needed, Magento finds all di.xml files and merges them together.
		Using a configuration file is easy: create that file in the /etc/ folder. Ideally, the file can be limited to a specific area, such as the frontend or adminhtml. Copy an existing file from another module to start with a boilerplate that works.
		You can also create custom configuration files. Several updates are required to do this:
			• A XSD defining the schema to validate configuration files.
			• VirtualType created that extends \Magento\Framework\Config\Reader\Filesystem, specifies the converter, schemaLocator, and fileName.
			• A Converter class that implements \Magento\Framework\Config\ConverterInterface
			• A SchemaLocater class that implements \Magento\Framework\Config\SchemaLocatorInterface
			• A Data class that extends Magento\Framework\Config\Data

	Which configuration files correspond to different features and functionality?

		Mostly detailed above. Some less popular files:
			• address_formats.xml: the output types for an address
			• extension_attributes.xml: generic entity extensibility system.
			• product_options.xml: types of product options (text, file, select, etc.)
			• product_types.xml: stores product types (simple, configurable, etc.)

##DEMONSTRATE HOW TO USE DEPENDENCY INJECTION
	
	Magento is very customizable. Magento’s Dependency Injection concept embraces that and allows a great deal of control.
	Dependency Injection is literally injecting what a class’ dependencies are into the constructor or setter methods.
	Alan Kent has a great article about dependency injection and its benefits.
	The beauty of dependency injection is that it is very easy to see what your class needs are at a glance. You build your class around the class or interface that you inject. You do not care what class or interface is injected.

	Describe Magento’s dependency injection approach and architecture.

		Magento’s dependency injection framework is unique in that it is very automatic. Many other frameworks require at least some level of configuration to get going. Magento provides ways to customize and adjust dependency injection on the fly as well.
		Magento uses constructor injection: that is, all of the dependencies are specified as arguments in the __construct() function.
		Before we continue, I should note that it is very poor practice to directly use the ObjectManager—the primary class that handles dependency inject. It is against Magento standards to do this (exception for only a few cases). 

	We specified a constructor for the above class. We are loading a reference entry to the Registry class. This happens by setting the class type and an argument name on the constructor.		
	Magento’s DI container holds a list of objects. Each time one is created, it is added into this container. Whenever an object is requested, it is loaded from this container.  In PHP, objects are references. Unless you clone the object, anywhere you pass the object (as an argument) that same object is referenced.
	For objects that might be time intensive to create, Magento provides proxies. A proxy lazy loads the class. To utilize this, specify a class like \Magento\Catalog\Model\Product\Proxy in di.xml. Proxies must NOT be specified in the __ constructor method.
	For objects that need to be new every time they are used, Magento uses factories. 
	To use a factory, specify the class or interface name and append Factory to the end. Like: \Magento\Catalog\Api\Data\ProductInterfaceFactory. Magento will find the preference for the ProductInterface (default is \Magento\Catalog\Model\Product) and create a factory for that class. The factory has one public method, create. Calling this method creates a new instance of the desired class. Magento creates these classes automatically. If you want, you can create factory classes. Here is an example: vendor/magento/module-paypal/Model/IpnFactory.php.
	For objects that might be time intensive to load, Magento provides proxies. A proxy lazy loads the class. To utilize this, specify a class like \Magento\Catalog\Model\Product\Proxy in the constructor.


	How are objects realized in Magento?
		Since dependency injection happens automatically in the constructor, Magento must handle creating classes. As such, class creation either happens at the time of injection or with a factory.

	Class creation at the time of injection
		A great way to watch this step-by-step is to set a breakpoint in vendor/magento/framework/App/Router/Base.php::matchAction, on the line that contains $this->actionFactory->create().
		The first step in the process is the object manager locating the proper class type. If an interface is requested, hopefully an entry in di.xml will provide a concrete class for the interface (if not, an exception will be thrown).
		The deploy mode (bin/magento deploy:mode:show) determines which class loader is used.
			• Developer: vendor/magento/framework/ObjectManager/Factory/Dynamic/Developer.php
			• Production: vendor/magento/framework/ObjectManager/Factory/Dynamic/Production.php

		The parameters for the constructor are loaded. Then those parameters are recursively parsed. Not only are the dependencies for the initially requested class loaded, but dependencies of dependencies as well.
		A metaphor of this would be a tree. In a tree, you have the trunk and then the branches. The trunk would represent an object type. But that object has dependencies, which continue splitting and going up the tree. Eventually, you have all the branches and all the leaves representing all of the classes (dependencies) that your class needs to perform its functions.

	Class creation with a factory
		There are some instances where you do not want a shared object. A good example would be when creating new product and persisting that to the database. If you requested an instance of \Magento\Catalog\Api\Data\ProductInterface, that object will be shared with all other requests for that instance. As such, calling any setter methods on this object result in that value being change in all other requests for that instance.
		If an inject object does not store data in the class (ie. all data is transient and only passes through the class), you likely do not need to load an instance of the class with a factory. If you need to use a class that stores data in the class (for example in the _data array), you likely need to use a factory.
		Another way of looking at it is instead of calling new ClassName() in the code, inject an instance of ClassNameFactory into the constructor and call $classNameFactory->create() instead.To see Magento auto-created factories, look in the /generated folder. If you need to create a custom factory, feel free to copy one of these as boilerplate, changing the namespace, class name and likely the parameters of the create function.


	Why is it important to have a centralized process creating object instances?
		Having a centralized process to create objects makes testing much easier. It also provides a simple interface to substitute objects as well as modify existing ones.

	Identify how to use DI configuration files for customizing Magento.
		You should be very familiar with di.xml and how to use it.

	Plugins
		Plugins allow you to wrap another class’ public functions, add a before method to modify the input arguments, or add an after method to modify the output. These will be discussed more in the next section.Example: vendor/magento/module-catalog/etc/di.xml (search for “plugins”)

	Preferences
		Preferences are used to substitute entire classes. They can also be used to specify a concrete class for an interface. If you create a service contract for a repository in your /Api folder and a concrete class in /Model, you can create a preference.

	Virtual Types
		A virtual type allows the developer to create an instance of an existing class that has custom constructor arguments. This is useful in cases where you need a “new” class only because the constructor arguments need to be changed.
		This is used frequently in Magento to reduce redundant PHP classes.

	Argument Preferences / Constructor Arguments
		It is possible to modify what objects are injected into specific classes by targeting the name of the argument to associate it with the new class.
		Now, with Magento 2, you can inject your custom class into any other classes constructor in di.xml

	
	How can you override a native class, inject your class into another object, and use other techniques available in di.xml (such as virtualTypes)?

		Overriding a native class: use a <preference/> entry to specify the existing class name (preceding backslash \ is optional) and the class to be overridden.
		Inject your class into another object: use a <type/> entry with a <argument xsi:type="object">\Path\To\Your\Class</argument> entry in the <arguments/> node
		Other uses of di.xml are found above.

###DEMONSTRATE ABILITY TO USE PLUGINS
