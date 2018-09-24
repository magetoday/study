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

