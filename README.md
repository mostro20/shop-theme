# RWAHS Magento2 Shop Theme

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

## Prerequisites

Please see the following link for the Magento system requirements.  Installation of system requirements is outside the
scope of this document.  Composer is required.

* http://devdocs.magento.com/guides/v2.0/install-gde/system-requirements.html

## Note about the Nginx sample configuration

Magento2 ships with an `nginx.conf.sample` file, which is mostly correct.  However, it sets the `root` to 
`$MAGE_ROOT/pub`, which triggers an error when importing images (tested on `2.0.8`).  See bug report at 
https://github.com/magento/magento2/issues/6449.

The workaround is to set the web server root to `$MAGE_ROOT` instead of `$MAGE_ROOT/pub`.

## Admin UI Locale Configuration

* Magento2 serves static content (stylesheets and javascript) on a per-locale basis.
* The database seed sets the default locale to `en_AU`.  This means that by default (and as per the instructions below)
  the static content is only built for the `en_AU` locale; if any other locale is used, the page will be broken.
* When an admin user is logged in, their profile determines the locale used.  All users should always use `en_AU`.
* The locale can also be set by appending a URL parameter: `?locale=en_AU`; this is saved as a cookie and used for 
  future requests.
* The login screen does not use the default locale, it always uses `en_US` by default.  Therefore the distributed login
  URL should always have the `?locale=en_AU` appended.

## Steps

### 1. Clone repositories

```
mkdir -p /srv/deploy
cd /srv/deploy
git clone https://github.com/rwahs/shop-theme.git
git clone https://github.com/rwahs/shop-migration.git
```

You will also need to setup a `local-settings` file in the `shop-migration` clone, as described by its README file.

### 2. Install Magento code

* Version: 2.0.x
* Method: composer

Example to install `2.0.8`:

```
cd /srv/sites/documentation-shop.histwest.org.au
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition . 2.0.8
```

**Note** The remainder of the following instructions assume that you are following them in order, and have therefore 
already changed directory to the Magento installation root as per the first command above.

**Note** If the version is omitted, the latest stable release is installed, which may yield unexpected results.  The 
database seed and theme files were created for and tested against 2.0.x only.

**Note** To run the `composer` installation, you will also need an access key as described here:
 
* http://devdocs.magento.com/guides/v2.0/install-gde/prereq/connect-auth.html

### 3. Set file permissions

The following page describes the file system ownership requirements for a multi-user system, where the magento user is
separate from the web server user:

* http://devdocs.magento.com/guides/v2.0/install-gde/prereq/integrator_install.html#mage-owner-about-group

These are the commands to run (taken from the above link):

```
find var vendor pub/static pub/media app/etc -type f -exec chmod g+w {} \;
find var vendor pub/static pub/media app/etc -type d -exec chmod g+ws {} \;
chown -R :www-data .
chmod u+x bin/magento
```

### 4. Run the Magento installer

This step is required to configure the database connection and backend URL; other settings will be overwritten by 
installing the database seed in the next step.

```
bin/magento setup:install --db-host=documentation-shop-db.histwest.org.au --db-name=shop --db-user=magento2 --db-password=password --backend-frontname=rwahs_admin --admin-user=administrator --admin-password=password1 --admin-email=rwahs@gaiaresources.com.au --admin-firstname=System --admin-lastname=Administrator 
```

### 5. Copy the theme

There is an issue with building the theme using a symlink, so we must copy the files under `app/design/frontend`.

```
mkdir -p app/design/frontend/Gaia
cp -R /srv/deploy/shop-theme app/design/frontend/Gaia/rwahsluma
```

### 6. Install the database seed

The database seed contains most of the custom configuration, theme selection, administrator details, etc.

```
/srv/deploy/shop-migration/bin/reset-db
```

### 7. Set Magento to `production` mode

In any deployed environment, Magento should be in `production` mode, as opposed to `default` or `developer`.  

```
bin/magento deploy:mode:set production
```

### 8. Install procurement management plugin

See also: http://documentation.boostmyshop.com/inventory_management_magento2/2_installation.html however note that this
refers to a different plugin with additional steps (database upgrade) that are not required here.  The plugin zip file 
should be unpacked to a known location, e.g. `/path/to/bms-magento2-supplier-0.0.8`.

```
cp -R /path/to/bms-magento2-supplier-0.0.8/app/code app/
chown -R magento2:www-data app/code
bin/magento setup:upgrade
rm -rf var/generation var/di
bin/magento module:enable BoostMyShop_Supplier
bin/magento setup:di:compile
```

### 9. Move media from the database to the file system

1. Log in to the admin user interface at: https://documentation-shop.histwest.org.au/rwahs_admin?locale=en_AU
2. Go to: Stores - Configuration - Advanced - System - Storage Configuration for Media.
3. Change `Media Storage` to `File System`, then click `Synchronize` and then `Save Config` (top right).
4. Flush the cache, either by going to System - Cache Management and clicking `Flush Magento Cache`, or by executing 
   `bin/magento cache:flush`.

The RWAHS shop is now fully operational.
