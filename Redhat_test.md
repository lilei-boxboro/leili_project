# Getting Started With ownCloud #

This Getting Started Guide describes a manual Linux installation. For other types of installations, see the Table of Contents on the [Download ownCloud page](https://owncloud.org/download/). This Getting Started Guide describes how to do the following.

- Install and configure an ownCloud server.
- Add a user account.
- Enable users to connect to the ownCloud server using a desktop or mobile client. 

## Installing and Configuring an ownCloud Server ##

### Before You Begin ###

- Log in with Administrator privileges.
- Ensure that [supported PHP extensions](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#prerequisites) (version 7.2 or later) are installed, enabled, and configured on your Linux system.- 
- Depending on the version of Linux you are running, install the [required packages, modules, and extensions](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#ubuntu-installation-label) needed to support ownCloud.

### Install ownCloud

Install ownCloud by performing the following steps.

1. Browse to the [Download ownCloud](https://owncloud.org/download/) page.
2. Navigate to the section **Resources > Tarball** and download either the tar or zip archive. The downloaded file is named owncloud-*x*.*y*.*z*.tar.bz2 or owncloud-*x*.*y*.*z*.zip (where *x*.*y*.*z* is the version number).
3. Download the corresponding checksum file for validation. For example, owncloud-*x*.*y*.*z*.tar.bz2.md5, or owncloud-*x*.*y*.*z*.tar.bz2.sha256, (where *x*.*y*.*z* is the version number).
4. Verify the MD5 or SHA256 sum, using the following commands as examples.     

	 
	``md5sum -c owncloud-x.y.z.tar.bz2.md5 < owncloud-x.y.z.tar.bz2``

	``sha256sum -c owncloud-x.y.z.tar.bz2.sha256 < owncloud-x.y.z.tar.bz2``

    ``md5sum  -c owncloud-x.y.z.zip.md5 < owncloud-x.y.z.zip``

    ``sha256sum  -c owncloud-x.y.z.zip.sha256 < owncloud-x.y.z.zip``
	

1. Verify the PGP signature for authenticity, using the following commands as examples.

     
    ``wget https://download.owncloud.org/community/owncloud-x.y.z.tar.bz2.asc``

    ``wget https://owncloud.org/owncloud.asc``

    ``gpg --import owncloud.asc``

    ``gpg --verify owncloud-x.y.z.tar.bz2.asc owncloud-x.y.z.tar.bz2``
  
1. Extract the archive contents by running the appropriate  command for your archive type, unpacking the archive to a single ownCloud directory. Use the following commands as examples.

	``tar -xjf owncloud-x.y.z.tar.bz2``

	``unzip owncloud-x.y.z.zip``


1. Copy the ownCloud directory to the destination. If you are running the Apache HTTP server, you install ownCloud in your Apache *document root* by entering the following command, where */path/to/webserver/document-root* is the path to your Web server.  

	``cp -r owncloud /path/to/webserver/document-root``

	For example: ``cp -r owncloud /var/www``

	*Note: On other HTTP servers, install ownCloud outside of the document root.*

### Configure Apache Web Server


1. On Debian, Ubuntu, and their derivatives, create a ``/etc/apache2/sites-available/owncloud.conf`` file with these lines in it, replacing the *Directory* and other file paths with your own file paths:

	``Alias /owncloud "/var/www/owncloud/"``
	
	``<Directory /var/www/owncloud/>``
	
	``Options +FollowSymlinks``
	
	`AllowOverride All``
	
	``<IfModule mod_dav.c>``
	
	``Dav off``
	
	`` </IfModule>``
	
	``SetEnv HOME /var/www/owncloud``
	
	``SetEnv HTTP_HOME /var/www/owncloud``
	
	``</Directory>``

1. Create a symbolic link to ``/etc/apache2/sites-enabled`` by entering the following command.

	``ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf``

1. Enable required modules by running these commands.

    ``a2enmod rewrite``

    ``a2enmod headers``

    ``a2enmod env``

    ``a2enmod dir``

    ``a2enmod mime``

*Note: See [Additional Apache Configurations](https://doc.owncloud.org/server/10.0/admin_manual/installation/source_installation.html#install-owncloud)  for additional considerations based on your installation.*

### Enable SSL

We strongly recommend using SSL/TLS to encrypt all of your server traffic, and to protect user log-ins and data in transit.

Enable Apache's ``ssl`` module and default site by running the following in a terminal window:

``a2enmod ssl``
``a2ensite default-ssl``
``service apache2 reload``

### Run the Installation Wizard

1. Restart the Apache server.
2. Run the [Installation Wizard](https://doc.owncloud.org/server/10.0/admin_manual/installation/installation_wizard.html) with GUI or the command line (using the ``occ`` command). If you use the command line, see **Run the Installation Wizard** on the ownCloud site for details about changing and setting permissions, managing trusted domains among other important considerations.  

### Configure the ownCloud Server

Although you do not need to customize the server configuration file, you may choose to do so in the event that you want to specify a specific port for ownCloud to listen for requests from clients.

To specify a particular IP address and port number to connect directly to ownCloud, do the following.

1. Open the ownCloud server configuration file for editing. On Linux, this file is located at:  ``HOME/.config/ownCloud/owncloud.cfg``.
2. In the **Proxy** section, set the following values:
	- For **Host**, specify the IP address of the server that users will connect to, for example **127.0.0.1**.
	- For **Port**, specify a secure port, such as **8080**. 
	- For **Type**, specify **2**, indicating that this server is not being used as an explicit proxy.
2. Save and close the configuration file.

See the section **Enable Users to Connect to the ownCloud Server** in this Getting Started Guide for steps to connect from either a desktop or mobile client.

*Note: See the [ownCloud Server Configuration page](https://doc.owncloud.org/server/10.0/admin_manual/configuration/server/index.html) for a complete list of configuration tasks.*

## Add a User Account

Define a new user to the ownCloud server by entering the following command. 

	user:add [--password-from-env] [--display-name [DISPLAY-NAME]] 
      [--email [EMAIL]] [-g|--group [GROUP]] [--] <uid>


	
where:

-	*password-from-env* specifies that the user's password be the same as their domain password.
- 	*display name* is the user's  full name that appears on the User's page in the ownCloud Web user interface.
- 	*email address* is the user's email address.
- 	*group* is the group name to which the user belongs.
- 	*uid* is user’s user name and their log-in name.
	
See **User Management** in the [ownCloud Server Configuration page](https://doc.owncloud.org/server/10.0/admin_manual/configuration/user/index.html) for a complete list of additional arguments, examples of user commands and other user management tasks.

## Enable Users to Connect to the ownCloud Server

Users connect to the ownCloud server using either a desktop or mobile client.

### Connecting with a Desktop Client

To connect to ownCloud with a desktop, install the [Desktop Synchronization Client](https://doc.owncloud.org/desktop/2.5/installing.html). After doing so, users can launch the client, log-in, and synchronize, upload, and download files.

### Connecting with a Mobile Client

To connect to ownCloud with a mobile device, install the [ownCloud mobile application](https://owncloud.com/apps/).  After doing so, users can log-in and synchronize, upload, and download files from any compatible mobile device including phones and tablets.

Optionally, use your browser and connect using URL or IP. For example:

``https://www.example.org/owncloud``
``https://127.0.0.1/owncloud``


For detailed information on using the clients, see the [ownCloud Client User Manual](https://doc.owncloud.org/server/8.2/user_manual/files/access_webdav.html).


## For More Information About ownCloud

See the [ownCloud documentation site](https://doc.owncloud.org/server/index.html) for comprehensive task and reference material about ownCloud.
