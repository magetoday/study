#Magento Architecture


##DESCRIBE MAGENTO’S MODULE-BASED ARCHITECTURE

	POINTS TO REMEMBER
		• There are five areas: adminhtml, frontend, base, webapi_rest, webapi_soap and cron. Not all areas are always available. For instance, the cron area is only used when	running cron jobs.
		• The three necessary files to bootstrap a module are registration.php, etc/module.xml and composer.json. 

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

	POINTS TO REMEMBER
		• Dependency injection is a means of giving a class what it needs to function.
		• ObjectManager is Magento’s internal object storage unit and should rarely be directly accessed.
		• The ObjectManager makes implementing the Composition over Inheritance principle possible.
		• Dependency injection makes testing easier, application more configurable and provides options for powerful features such as plugins.


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

##DEMONSTRATE ABILITY TO USE PLUGINS

	POINTS TO REMEMBER
		• There are three types of plugins: around, before and after.
		• Plugins only work on public methods.
		• They do not work on final methods, final classes.
		• They must be configured in di.xml.
		• Important: plugins can be used on interfaces, abstract classes or parent classes. The plugin methods will be called for any implementation of those abstractions.

	Demonstrate how to design complex solutions using the plugin’s life cycle.

		Keeping methods to a small number of lines of code is sometimes challenging.
		There are those methods that seem to have to do everything.
		Magento 2’s idea of plugins brings a completely new idea to the table. Every public method can be intercepted, changed, or even circumvented.
		There are three types of plugins: around, before, and after. There are some complications with the around plugin, so it is advised to use it sparingly and only when the others will not do. Before plugins modify the input arguments to a method. You can change them to any value. After plugins are used to modify the return value. 
		Let’s say you have a complex saving operation. In this operation, you also need to validate the input data and return an error when there is a problem. You also need to respond with something more than true or false.
		You can use an around plugin to validate the incoming request and cancel the save if the result is invalid. Doing this, you are applying the single responsibility principle, making your code easier to understand, debug, and test.
		While plugins are often thought of as modifying core functionality, that example demonstrates that they can be useful for a broad range of applications.

	How plugins work
		When you create a plugin entry, Magento automatically generates a class wrapper for the plugin target. For example, if you want to modify \Magento\Catalog\Model\Product, Magento will auto-generate the \Magento\Catalog\Model\Product\Interceptor class. Every function inside the target class will be represented in the auto-generated interceptor class (if you add new functions to a target class, you may need to delete the auto-generated interceptor class from the /generated folder).
		Magento then handles locating the plugins and executing them in the Interceptor class: vendor/magento/framework/Interception/Interceptor.php

	Before Plugin
		Example from: \Magento\Catalog\Block\Product\ListProduct
		If you want to modify the input arguments of a method, create a before plugin. To modify the prepareSortableFieldsByCategory($category) method, add a method to the plugin class.
		The method above is run before \Magento\Catalog\Block\Product\ListProduct::prepareSortableFieldsByCategory. The return value for the before plugin determines the arguments going into the next plugin or the final targeted method.

	After Plugin
		Example from: \Magento\Catalog\Block\Product\ListProduct

		If you need to modify the output from a public method, use an after plugin. In our example class, let’s modify the getProductPrice($product). As such in our plugin class, we would create.
		The after plugin (as of 2.2) includes the input parameters in addition to the return result.

	Around Plugin
		The around plugin provides full control over the input and output of a function. The original function is passed in as a callback and, by standard, is named $proceed. Magento recommends against using these plugins whenever possible. This is because it is easy to accidentally alter major functions in the system by omitting a call to $proceed(). It also adds frames to the call stack making debugging more cumbersome.

	How do multiple plugins interact, and how can their execution order be controlled?
		With every good idea come potential downsides. Controlling how multiple plugins interact would be the problem with plugins. When a plugin is declared, the sortOrder attribute can be set. The lower the sort order, the sooner it will be executed in the list. The greater the sort order, the later it will be executed. This allows a degree of control over how one plugin will interact with others.
		Additionally, if you need to disable an existing plugin, you can reference it by the name attribute and add the disabled attribute.

	How do you debug a plugin if it doesn’t work?
		First unplug it, then remove the bug.
		There are a number of things that can go wrong with plugins. Here are some things to check:
			• Is the di.xml configuration correct? Are there any syntax errors?
			• Is the plugin marked as disabled?
			• Do you have the correct class specified in the <type name="..."> node? Is it the target class?
			• Do you have the correct plugin class specified in the <plugin type="..."> node?
			• Is the class or method you are modifying marked as final? If so, plugins will	not work.
			• Does your plugin class have a method to modify a method on the target class?
			• beforeMethodName or afterMethodName or aroundMethodName
			• NOT methodName
			• Do any of the limitations mentioned in the DevDocs apply? 

	One technique that has been helpful for us is to set a breakpoint in the method you want to debug. When that breakpoint has been encountered, look at the call stack to see if there are any references to an Interceptor class in the recent call stack.

	Identify strengths and weaknesses of plugins.
		Plugins are very powerful to discreetly modify functionality of existing code. They can also be used to follow the single responsibility principle (SRP) by segregating each piece of functionality to their own areas.
		The greatest weakness is exploited in the hands of a developer who is either not experienced or not willing to take the time to evaluate the fallout. For example, used improperly, an around plugin can prevent the system from functioning. They can also make understanding what is going on by reading source code hard (spooky action at a distance).

	What are the limitations of using plugins for customization?
		• Plugins only work on public functions (not protected or private).
		• Plugins do not work on final classes or final methods.
		• Plugins do not work on static methods.
		• Read more: http://devdocs.magento.com/guides/v2.1/extension-dev-guide/plugins.html

	In which cases should plugins be avoided?
		Plugins are useful to modify the input, output, or execution of an existing method. 
		Plugins are also best to be avoided in situations where an event observer will work. 
		Events work well when the flow of data does not have to be modified.

##CONFIGURE EVENT OBSERVERS AND SCHEDULED JOBS

	POINTS TO REMEMBER
		• Event observers listen to events that are triggered within Magento.
		• Event observers should not modify the sent data (what plugins are for).

		Event observers and scheduled jobs are used to carry out tasks on data. They are an ideal way to extend Magento functionality.

		Event observers and scheduled jobs carry a similar characteristic: both do not (should not) modify data as it traverses event observers. A scheduled job makes modifying the flow of data impossible while event observers still do allow it (even though it is against Magento development guidelines).
		When an action occurs, an event can be triggered. Event observers listen to these events and act as a notification system. Observers implement \Magento\Framework\Event\ObserverInterface.

		If you need to modify the data in a method, it is best to use a before or after plugin.

	Demonstrate how to configure observers.
		To create an event observer, create the file events.xml in the etc directory. If the event only needs to be listened to in a specific area, create an events.xml in that directory.
		Create a class that will receive the payload from the event dispatcher. This class must implement \Magento\Framework\Event\ObserverInterface.

	How do you make your observer only be active on the frontend or backend?
		Place it in the /etc/[area]/events.xml folder.

	Demonstrate how to configure a scheduled job.
		To execute a specific action on a schedule, you need to setup the crontab.xml file. This file always resides in the /etc folder (not in /etc/[area]).

		Group
			Magento allows you to group cron activity together, making logical groups of functionality. For most scheduled activity, use the default group. Magento also does provide index group. You can set up configuration options for groups in cron_groups.xml.

		Job
			Configuring the job is simple:
				• assign a unique name
				• specify the class
				• define a method
				• set a schedule (using regular crontab schedule notation)

	Which parameters are used in configuration, and how can configuration interact with server configuration?
		System environment variables can be used to change store configuration information. To change a store configuration value with environment variables:
			1. Find the store configuration path found in the database, etc/adminhtml/system.xml or the admin panel.
			2. Change “/” to “__” (double underscores) in the path.
			3. If this configuration will be set on a global basis (not per store), prepend the path with CONFIG__DEFAULT__.
			4. If this configuration will be set for a particular website, prepend the path with CONFIG__WEBSITES__[WEBSITE_CODE]__

	Identify the function and proper use of automatically available events, for example *_load_after, etc.

		Magento provides a number of pre-built events. These cover functionality for request routing, data loading, caching, and HTML output.

		One of the most common uses for event listeners / observers is with data loading. For your reference, here is a list of all events triggered in \Magento\Framework\Model\AbstractModel (the parent class of data that is loaded from the database):
			• $this->_eventPrefix . _load_before
			• $this->_eventPrefix . _load_after
			• $this->_eventPrefix . _save_before
			• $this->_eventPrefix . _save_after
			• $this->_eventPrefix . _save_commit_after
			• $this->_eventPrefix . _delete_before
			• $this->_eventPrefix . _delete_after
			• $this->_eventPrefix . _delete_commit_after
			• $this->_eventPrefix . _clear

		$this->_eventPrefix is a protected variable that is set in a model (see vendor/magento/module-catalog/Model/Product.php for an example).


	When to use events or plugins?
		At a fundamental level, the data that is sent to events should not be transformed. 
		Events should be able to be run completely asynchronously:
			• You can use the catalog_product_save_commit_after to refresh a static file that you have generated. You could use save_after—the downside is if an uncaught exception is thrown in the observer, the database transaction will be rolled back and data will not be saved. As long as you have product data, this operation can run at any future date.
			• You can send a notification email when an action has occurred (similar to sending an email when an order is placed).

		 If you want to transform data, use a plugin:
		 	• If you want to transform data, the Magento technical requirements say to use a plugin. The reason for this is that plugins are designed to modify the flow of data into and out of a method. They wrap and control the data flow. Events could be viewed as the spoke of a bicycle. The data is dispatched to the event listeners. There is no expectation of an order of processing and the data could be handled in different orders at different times.
		 	• That said, events are still widely used even for data mutations. 
		 	• If you want to modify data before a product is saved, create an after plugin for the beforeSave method: afterBeforeSave (a little hard to read).
		 	• If you need to add some additional data when a customer has been loaded, you can do this as an after plugin in the afterLoad method: afterAfterLoad. Ideally, you would be using extension attributes for this purpose as well.

		 To dispatch an event, inject an instance of \Magento\Framework\Event\ManagerInterface into the constructor. Then call:
		 	$this->eventManager->dispatch('event_name_goes_here', ['parameter' => 'array']);

##UTILIZE THE CLI

	POINTS TO REMEMBER
		• The Magento CLI is based on the Symfony Console component.
		• It is important to familiarize yourself with these commands.

	Describe the usage of bin/magento commands in the development cycle.
		The Magento CLI is the Magento developer’s good friend. It contains many commands to make your life easier.

		Hint: create an alias in .bash_profile like: alias mage="php -d memory_limit=1024M bin/magento". With this, you can enter a Magento 2 directory and type "mage setup-upgrade" instead of typing "bin/magento setupupgrade".

		The Magento CLI is based on the Symfony Console component. If you are interested in building CLI commands, reference this document for helpful information. Additionally, Magento DevDocs provides a tutorial on how to add commands.
		These next topics will discuss some of the most useful CLI commands in the development process. Note the abbreviations next to some of the commands: this is a feature in Symfony Console.
		To see the available options for a command, run the command and add --help to the end.

	bin/magento cache:flush (abbreviated bin/magento c:f)
		Flushes, or destroys, the cache. It is different than cache:clean in that flush actually deletes the keys. The end result is typically the same.

	bin/magento cache:status
		Lists the cache types and their status (enabled / disabled). This is helpful for toggling caches with the cache:enable / cache:disable command.

	bin/magento deploy:mode:show
		This command shows the current deploy mode: developer, default, or production. When developing Magento, your life will be much easier if you turn the deploy mode to “developer.”

	bin/magento dev:query-log:enable
		Turns on logging of database queries. Can be helpful in tracking down an obscure bug. My preference is to use xdebug and set a breakpoint in vendor/magento/framework/Model/ResourceModel/Db/AbstractDb.php as logging can be verbose.

	bin/magento indexer:info AND bin/magento indexer:reindex
		These commands show indexer details or initiate a reindex respectively. 

	bin/magento module:enable
		After creating the required module files, you need to enable the module in Magento 2 (Magento 1 did this automatically for you). This is done by running the above command with either the --all flag (enabling all modules) or with appending the name of the module to enable to the above command.

	bin/magento module:status
		Lists the enabled and disabled modules. If you are having trouble figuring out why a module is not working, this is a good command to put into your troubleshooting repertoire. This has been very helpful to me because it is easy to forget to enable modules.

	bin/magento setup:upgrade
		When the version of a module (in etc/module.xml) is incremented, you need to run this command to synchronize those changes to the database. Until you do so, your entire frontend and admin panel will show an error message.
		You can run this command with a --keep-generated flag to prevent the default behavior of removing all generated assets. You can use this flag in a code deploy situation where your assets have already been compiled for the new version.

	bin/magento setup:db:status
		To check the upgrade status for Magento, use this command. Check the output of this command to determine whether or not you need to upgrade the database (using setup:upgrade).

	Which commands are available?
		To see what commands are available, execute bin/magento in your command line. 
		This provides a list of all commands available.

	As of Magento 2.2.3 (March 2018), here is the current list of commands for Magento Open Source:
		Image follow:


	How are commands used in the development cycle?
		CLI commands provide a secure entry point for conducting operations that could be insecure to run in the Magento admin panel. SSH access should be a secure way to vet a user’s authorization status.
		Magento provides commands to run tests (dev:tests:run although using PHPUnit in PHPStorm is faster and provides xdebug capabilities), compile dependency injection (setup:di:compile), and deploy static assets (setup:static-content:deploy).
		Enabling modules and running setup scripts has to be done using the command line during development. It is often quicker to toggle caching of sections or to flush the cache during development using the command line compared to using the admin interface.

	Demonstrate an ability to create a deployment process.
		Deploying Magento 2 involves several steps. 
		Publisher’s note: we would like to eventually publish some articles on a build > deploy system that we use at SWIFTotter. Look on our website to see these articlesas we are able to release them.

		The basic steps for deploying Magento 2 code:
			• Enable maintenance mode
			• Copy files to the deploy target
			• Enable modules / apply deploy configuration
			• Run dependency compilation
			• Build static assets
			• Upgrade the database
			• Disable maintenance mode


		As you can see, this process involves downtime for production. For many companies, up to 30 minutes of downtime each deploy is not acceptable. Magento has been actively working to mitigate this. In addition, several community systems are available. Look in the further reading section below.

	How does the application behave in different deployment modes, and how do these behaviors impact the deployment approach for PHP code, frontend assets, etc.?
		Default (not ideal for production or development, but as a hybrid)

		• Symlinks are established in the pub/static folder. These are linked from files in the app/code or app/design folders.
		• Exceptions are not displayed to the user (making development very difficult). They are logged in var/log.
		• Static files are generated on the fly and are symlinked into the var/view_ preprocessed folder.

	Developer mode (security risk if used in production)
		• Symlinks are established in the pub/static folder. Use dev:staticcontent:deploy to refresh these links or simply remove the stale link manually (much faster).
		• Errors are shown to the user and logging is verbose. Caution: debug logging is disabled by default even in developer mode.
		• Magento automatically builds code for plugins (interceptors), factories, etc. as it does in the other modes.
		• Slow performance.

	Production mode (difficult to use for development)
		• Static files must be pre-compiled as no compilation will happen on the fly.
		• Errors are only logged.

## DEMONSTRATE THE ABILITY TO MANAGE THE CACHE

	POINTS TO REMEMBER
		• Understanding the config, layout, block_html, config_ webservice and full_page caches and their functions is important.
		• You can disable full page caching on a particular page by marking a block as cacheable="false"

	Describe cache types and the tools used to manage caches.
		Magento includes multiple types of caching to speed retrieval of CPU-consuming calculations and operations. To see a list of all cache types, run bin/magento cache:status.

		Magento 2 includes two means of caching: server caching, Varnish caching and browser caching.

		Cache configuration is stored in /etc/cache.xml. Here is a list of all the cache.xml files in Magento 2.2:
			• module-eav/etc/cache.xml
			• module-translation/etc/cache.xml
			• module-customer/etc/cache.xml
			• module-webapi/etc/cache.xml
			• module-page-cache/etc/cache.xml
			• module-store/etc/cache.xml
			• module-integration/etc/cache.xml

		You can clear a specific cache by using the bin/magento cache:flush command. For example, the following clears the config and layout caches: bin/magento cache:flush config layout. Here is a list of some of the more important caches:
			
			config: Magento Configuration
				The config cache stores configuration from the XML files along with entries in the core_config_data table. This cache needs to be refreshed when you add system configuration entries (/etc/adminhtml/system.xml) and make XML configuration modifications.

			layout: Layout XML Updates
				With Magento’s extensive layout configuration, a lot of CPU cycles are used in combining and building these rules. This cache needs to be refreshed when making changes to files in the app/design and the app/code/AcmeWidgets/ProductPromoter/view/[area]/layout folders. For frontend development, we usually disable this cache. This cache type is also used to cache ui_component XML.

			block_html: Output from the toHtml method on a block
				Obtaining the HTML from a block can also be expensive. Caching at this level allows some of this HTML output to be reused in other locations or pages in the system. For frontend development, we usually disable this cache.

			collections: Multi-row results from database queries
				This cache stores results from database queries. 

			db_ddl: Database table structure
				See this file: vendor/magento/framework/DB/Adapter/Pdo/Mysql.php

			config_webservice:
				This stores the configuration for the REST and SOAP APIs. When adding methods to the API service contracts, you will need to flush this one frequently.

			full_page: Full page cache (FPC)
				This HTML page output can be stored on the file system (default), database or Redis (fastest). When doing any type of frontend development, it is best to leave the FPC off. Before deploying new frontend updates, though, it is important turn it back on and ensure that the updates do not cause problems with the cache.


		How do you add dynamic content to pages served from the full page cache?
			In Magento 2 FPC, pages are either cached or they are not. To make a page not cacheable, you can add the cacheable="false" attribute to any block on the page. Keep in mind that this affects website performance and should be used sparingly.
			Combining speed and personalization involves serving a cached page and then substituting or adding content with an AJAX request.
			The DevDocs contains an article on how to do this with UI Components. This is considered the only “proper” way of hole punching as Magento leverages the browser’s local cache storage system.

		Describe how to operate with cache clearing.
			When working with the cache, assume that the requested cache entry is not present. If it is not present, run your CPU-intensive operation, and save the results into the cache. Subsequent visits to the cache should be populated and save precious computation time.

		How would you clean the cache?
			bin/magento cache:clean OR bin/magento cache/flush

		In which case would you refresh cache/flush cache storage?
			Magento recommends running cache cleaning (cache:clean) operations first as this does not affect other applications that might use the same cache storage. If this does not solve the problem, they recommend flushing the cache storage (cache:flush).
			In reality, if the file system cache storage is used, you should never have multiple applications' cache storage combined. Sessions and content caching should never share a database in Redis. Ideally, they are stored in altogether separate Redis instances. As such, flushing the cache should not have any consequences.

		Describe how to clear the cache programmatically. 
			You can clear the cache programmatically by calling the \Magento\Framework\App\CacheInterface::remove() method.

		What mechanisms are available for clearing all or part of the cache?

			clean_cache_by_tags event
				You can dispatch a clean_cache_by_tags event with an object parameter of the object you want to clear from the cache.
				From: \Magento\PageCache\Observer\FlushCacheByTags::execute \Magento\Framework\App\CacheInterface->clean()
				As seen above in “Describe how to clear the cache programmatically.”

			Magento CLI console
				bin/magento cache/flush or bin/magento cache:clean

			Manually
				You can rm -rf var/cache/ or use the redis-cli, select a database index, and run flushdb.