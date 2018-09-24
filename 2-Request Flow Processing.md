#Request Flow Processing


##UTILIZE MODES AND APPLICATION INITIALIZATION
	
	POINTS TO REMEMBER
		• The recommended Magento entry point is pub/index.php
		• The default deploy mode is the default for Magento’s deploy mode feature.

	Identify the steps for application initialization.
	To see for yourself the path of execution, open vendor/magento/modulebackend/Controller/Adminhtml/Auth/Login.php and set a breakpoint in the execute() method. Clear cookies in your development website and navigate to the admin panel.

	Look at the call-stack for the execute() method:
		• The recommended application entry point is pub/index.php. Nginx or Apache should use /pub as the website’s primary directory.

		• pub/index.php