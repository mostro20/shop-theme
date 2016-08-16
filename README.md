# shop-theme

Magento2 theme for RWAHS shop, based on the pre-packaged Luma theme.

The following guide uses the following directory layout:

```
+ /
|
+-+ srv
  |
  +-+ deploy
  | |
  | +-- shop-theme (this repo is cloned here)
  | |
  | +-- shop-migration (the shop-migration repo is cloned here)
  |
  +-+ sites
    |
    +-- <environment>-shop.histwest.org.au (magento install path)
```

Where `<environment>` is the name of the environment being installed, e.g. `staging` or `production`.  In the commands below, the `<environment>` used is `documentation`.

The directories indicated above are used in commands below; if you want a different file system layout, you must alter the commands accordingly.

## Reference

These instructions follow the basic principles outlined in the Magento 2.0 documentation:

* http://devdocs.magento.com/guides/v2.0/install-gde/bk-install-guide.html

Note that no consideration is given in the instructions below to file system ownership and permission, however it is assumed that the Magento guidelines are adhered to:

* http://devdocs.magento.com/guides/v2.0/install-gde/prereq/integrator_install.html#mage-owner-about-group

Finally, server software is required:

* A web server; `nginx` is recommended, in combination with `php-fpm`.  Setup of the web server or CGI backend is outside of the scope of this document.
* A database server; `mysql` or a variant is required by Magento.  These instructions assume a database has been created and permissions assigned, as per standard procedure.

## Steps

### 1. Clone repositories

```
mkdir -p /srv/deploy
cd /srv/deploy
git clone https://github.com/rwahs/shop-theme.git
git clone https://github.com/rwahs/shop-migration.git
```

### 2. Install Magento

* Version: 2.0.x
* Method: composer

Example to install `2.0.8`:

```
composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition /srv/sites/documentation-shop.histwest.org.au 2.0.8
```

Note that if the version is omitted, the latest stable release is installed, which may yield unexpected results.  The database seed and theme files were created for and tested against 2.0.x only.

### 3. Install the Database Seed


### 4. Copy the Theme


### 5. Make the Theme Available to Magento


### 6. Set Magento to Production Mode



