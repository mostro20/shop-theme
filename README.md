# shop-theme

Magento2 theme for RWAHS shop, based on the pre-packaged Luma theme.

The following guide uses the following directory layout:

```
+ /
|
+-+ srv/
  |
  +-+ deploy/
  | |
  | +-- shop-theme/ (this repo is cloned here)
  | |
  | +-- shop-migration/ (the shop-migration repo is cloned here)
  |
  +-+ sites
    |
    +-- <environment>-shop.histwest.org.au/ (magento install path)
```

Where `<environment>` is the name of the environment being installed, e.g. `staging` or `production`.  In the commands 
below, the `<environment>` used is `documentation`.

The directories indicated above are used in commands below; if you want a different file system layout, you must alter 
the commands accordingly.

The database host is assumed to be `<environment>-shop-db.histwest.org.au`, the database name is `shop`, the database 
user is `magento2` and the password is `password`.  Alter these values as appropriate.

## Reference

These instructions follow the basic principles outlined in the Magento 2.0 documentation:

* http://devdocs.magento.com/guides/v2.0/install-gde/bk-install-guide.html

Note that no consideration is given in the instructions below to file system ownership and permission, however it is 
assumed that the Magento guidelines are adhered to:

* http://devdocs.magento.com/guides/v2.0/install-gde/prereq/integrator_install.html#mage-owner-about-group

Finally, server software is required:

* A web server; `nginx` is recommended, in combination with `php-fpm`.  Setup of the web server or CGI backend is 
outside of the scope of this document.
* A database server; `mysql` or a variant is required by Magento.  These instructions assume a database has been 
created and permissions assigned, as per standard procedure.

## Steps

### 1. Clone repositories

```
mkdir -p /srv/deploy
cd /srv/deploy
git clone https://github.com/rwahs/shop-theme.git
git clone https://github.com/rwahs/shop-migration.git
```

### 2. Install Magento code

* Version: 2.0.x
* Method: composer

Example to install `2.0.8`:

```
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition /srv/sites/documentation-shop.histwest.org.au 2.0.8
```

Note that if the version is omitted, the latest stable release is installed, which may yield unexpected results.  The 
database seed and theme files were created for and tested against 2.0.x only.

To run the `composer` installation, you will also need an access key as described here:
 
* http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html

### 3. Run the Magento installer

This step is required to configure the database connection and backend URL; other settings will be overwritten by 
installing the database seed in the next step.

```
cd /srv/sites/documentation-shop.histwest.org.au
bin/magento setup:install --db-host=documentation-shop-db.histwest.org.au --db-name=shop --db-user=magento2 --db-password=password --backend-frontname=rwahs_admin --admin-user=administrator --admin-password=password1 --admin-email=rwahs@gaiaresources.com.au --admin-firstname=System --admin-lastname=Administrator 
```

### 4. Copy the theme

There is an issue with building the theme using a symlink, so we must copy the files under `app/design/frontend`.

```
cd /srv/sites/documentation-shop.histwest.org.au/
mkdir -p app/design/frontend/Gaia
cp -R /srv/deploy/shop-theme app/design/frontend/Gaia/rwahsluma
```

### 5. Install the database seed

The database seed contains most of the custom configuration, theme selection, administrator details, etc.

```
/srv/deploy/shop-migration/bin/reset-db
```

### 6. Set Magento to `production` mode

The explicit call to `setup:static-content:deploy` is required due to requiring multiple locales.  `en_AU` is used in
the frontend, and `en_US` is used for the backend admin UI.  The final command sets Magento to production mode, 
including compiling the dependency injections and other configuration.

```
cd /srv/sites/documentation-shop.histwest.org.au/
bin/magento setup:static-content:deploy en_AU en_US
bin/magento deploy:mode:set production
```

### 7. Move media from the database to the file system

1. Log in to the admin user interface at https://documentation-shop.histwest.org.au/rwahs_admin.
2. Go to: Stores - Configuration - Advanced - System - Storage Configuration for Media
3. Change `Media Storage` to `File System`, then click `Synchronize` and then `Save Config` (top right)
4. Flush the cache, either by going to System - Cache Management and clicking `Flush Magento Cache`, or by executing 
   `bin/magento cache:flush`.

The RWAHS shop is now fully operational.
