# ownCloud Quick Start Guide#

This guide provides information about how to install a standard (community) ownCloud edition for Linux. All components will be installed on a single machine. This type of installation is recommended for small groups or departments of up to 150 users. 

For other types of standard ownCloud installations, see [Download ownCloud page](https://owncloud.org/download/). 

For comparisons of the ownCloud editions, see [ownCloud Server or Enterprise Edition](https://owncloud.com/standard-or-enterprise/).

This guide describes how to do the following.

- Install and configure an ownCloud server
- Add a user account
- Connect to the ownCloud server using a desktop or mobile client


## Installing and Configuring an ownCloud Server ##

Perform the tasks in this section to install and configure an ownCloud Server.

### Deployment recommendations ###

Use one machine for running the application, web and database server, and for local storage. Use an existing LDAP or Active Directory server for authentication.

![deployment recommendations](.\deprecs-1.jpg)

#### Components ####
Use one server with at least 2 CPU cores, 16GB RAM, and local storage as needed.

#### Operating system ####
Use Enterprise-grade Linux distribution with full support from an operating system vendor. RedHat Enterprise Linux and SUSE Linux Enterprise Server 12 are recommended.

#### SSL configuration ####
Because the SSL termination is done in Apache, a standard SSL certificate must be installed.

#### Database ####
Use MySQL or MariaDB with the InnoDB storage engine, or PostgreSQL.

#### Backup ####
Install ownCloud, the ownCloud data directory, and database on a Btrfs filesystem. Make regular snapshots at desired intervals for zero downtime backups. Mount DB partitions with the "nodatacow" option to prevent fragmentation.

Alternatively, make nightly backups with service interruption. 

#### Authentication ####
User authentication by using one or several LDAP or Active Directory (AD) servers. See [User Authentication with LDAP](https://doc.owncloud.org/server/10.4/admin_manual/configuration/user/user_auth_ldap.html) for information on configuring ownCloud to use LDAP and AD.

#### Session management ####
Local session management on the application server. PHP sessions are stored in a temporary filesystem and mounted at the operating system-specific session storage location. You can find out where that is by running ``grep -R 'session.save_path' /etc/php5`` and then add it to the ``/etc/fstab`` file.

For example:

    echo "tmpfs /var/lib/php5/pool-www tmpfs defaults,noatime,mode=1777 0 0" >> /etc/fstab`.

#### Storage ####
Local storage

### Before you begin ###

- Log in with administrator privileges.
- Ensure that [supported PHP extensions](https://doc.owncloud.com/server/10.4/admin_manual/installation/manual_installation.html#required) (version 7.2 or later) are enabled and configured on your Linux system.- 
- Ensure [all other prerequisites](https://doc.owncloud.org/server/10.4/admin_manual/installation/manual_installation.html#prerequisites/) are met.

### Install ownCloud

Follow these steps to install ownCloud:

1. Go to the [Download ownCloud](https://owncloud.org/download/) page.
2. Under **Resources > Tarball**, download either the tar or zip archive. The downloaded file is owncloud-*x*.*y*.*z*.tar.bz2 or owncloud-*x*.*y*.*z*.zip (where *x*.*y*.*z* is the version number).
3. Download archive's corresponding checksum file, for example, owncloud-*x*.*y*.*z*.tar.bz2.md5, or owncloud-*x*.*y*.*z*.tar.bz2.sha256, (where *x*.*y*.*z* is the version number).
4. Verify the MD5 or SHA256 sum:     

    ``md5sum -c owncloud-x.y.z.tar.bz2.md5 < owncloud-x.y.z.tar.bz2``
    
    ``sha256sum -c owncloud-x.y.z.tar.bz2.sha256 < owncloud-x.y.z.tar.bz2``

    ``md5sum  -c owncloud-x.y.z.zip.md5 < owncloud-x.y.z.zip``

    ``sha256sum  -c owncloud-x.y.z.zip.sha256 < owncloud-x.y.z.zip``
	
5. Verify the PGP signature for authenticity:

    ``wget https://download.owncloud.org/community/owncloud-x.y.z.tar.bz2.asc``

    ``wget https://owncloud.org/owncloud.asc``

    ``gpg --import owncloud.asc``

    ``gpg --verify owncloud-x.y.z.tar.bz2.asc owncloud-x.y.z.tar.bz2``
  
6. Extract the archive contents to a single ownCloud directory by running the appropriate command for your archive type: 

    ``tar -xjf owncloud-x.y.z.tar.bz2``

    ``unzip owncloud-x.y.z.zip``

7. Copy the ownCloud directory to the appropriate directory. If you are running the Apache HTTP server, copy it to your Apache *document root* by using the following command:  

    ``cp -r owncloud /path/to/webserver/document-root``

Where */path/to/webserver/document-root* is the path to your Web server, for example: ``cp -r owncloud /var/www``

*Note: On other HTTP servers, install ownCloud outside of the document root.*

### Configure Apache web server


1. On Debian, Ubuntu, and their derivatives, create a ``/etc/apache2/sites-available/owncloud.conf`` file with these lines in it, replacing the *Directory* and other file paths with your own file paths:

    ``Alias /owncloud "/var/www/owncloud/"``
	
    ``<Directory /var/www/owncloud/>``  
    `` Options +FollowSymlinks``  
    `` AllowOverride All``
	
    ``<IfModule mod_dav.c>``  
    `` Dav off``  
    ``</IfModule>``
	
    ``SetEnv HOME /var/www/owncloud``  
    ``SetEnv HTTP_HOME /var/www/owncloud``
	
    ``</Directory>``

2. Create a symbolic link to ``/etc/apache2/sites-enabled``:

    ``ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf``

3. Enable required modules:

    ``a2enmod rewrite``  
    ``a2enmod headers``  
    ``a2enmod env``  
    ``a2enmod dir``  
    ``a2enmod mime``

4. Disable any server-configured authentication for ownCloud, because ownCloud uses Basic authentication internally for DAV services. 

5. When using SSL, take note of the `ServerName`. You should specify one in the server configuration, and one in the `CommonName` field of the certificate.

6. Restart Apache:

    ``service apache2 restart``


### Enable SSL

It is recommended that you use SSL/TLS to encrypt all of your server traffic, and to protect user logins and data in transit.

To enable Apache's ``ssl`` module and default site, in a terminal window, enter the following commands:

    a2enmod ssl

    a2ensite default-ssl

    service apache2 reload
	
### Run the installation wizard

After restarting the Apache server, run the [Installation Wizard](https://doc.owncloud.org/server/10.4/admin_manual/installation/installation_wizard.html) with GUI or the command line (using the ``occ`` command). To enable this, temporarily change the ownership on your ownCloud directories to your HTTP user:

    chown -R www-data:www-data /var/www/owncloud/  

### Enable users to connect to the Owncloud server

You may specify a specific port for ownCloud to listen for requests from clients by modifying the server configuration file.

To specify a particular IP address and port number to connect directly to ownCloud, follow these steps:

1. Open the ownCloud server configuration file. On Linux, this file is in:  ``HOME/.config/ownCloud/owncloud.cfg``.
2. In the **Proxy** section, set the following values:
	- For **Host**, specify the IP address of the server that users will connect to, for example **127.0.0.1**.
	- For **Port**, specify a secure port, for example **8080**. 
	- For **Type**, specify **2**, indicating that this server is not being used as an explicit proxy.
3. Save and close the configuration file.


## Add a User Account

To add a new user to the ownCloud server:

1. On the ownCloud Users page, enter the new user’s login name and e-mail.
Login names may contain letters (a-z, A-Z), numbers (0-9), dashes (-), underscores (_), periods (.) and at signs (@).

2. Optionally assign group memberships.

3. Click **Create**.
	
4. Optionally enter a user's full name if it is different than the login name.
	

## Connect to the ownCloud Server Using a Desktop or Mobile Client

Users connect to the ownCloud server using either a desktop or mobile client.

### Connecting with a desktop client

You can download the latest version of the ownCloud Desktop Synchronization Client from the [ownCloud download page](https://owncloud.com/download/#desktop-clients). 

* On Mac OS X and Windows, double-click the installer and follow the setup wizard to complete the installation. After the sync client is installed and configured, it will get updated automatically.

* On Linux systems, follow the instructions on the download page to add the appropriate repository for your Linux distribution, install the signing key, and use the package managers to install the desktop sync client. Ensure that password manager, such as GNOME Keyring or KWallet, is enabled, so that the sync client can log in automatically.

 
### Connecting with a mobile client

To connect to ownCloud with a mobile device, install the [ownCloud mobile application](https://owncloud.com/apps/).  After doing so, you can browse all of your ownCloud synced files, create and edit files, share files and folders with others, and keep the contents of those folders in sync across all of your devices. 


## What's Next

After the installation and initial setup of ownCloud is complete, you can start using and configuring ownCloud. See the [ownCloud documentation](https://doc.owncloud.org/server/index.html) for comprehensive task and reference information.
