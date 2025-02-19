---
title: Configure the [!DNL Data Migration Tool]
description: Learn about the two methods for configuring the [!DNL Data Migration Tool] to transfer data between Magento 1 and Magento 2.
exl-id: 273be997-8085-4488-a455-f6005a85b406
---
# Configure the [!DNL Data Migration Tool]

After you install the [!DNL Data Migration Tool], the following directory contains mapping and configuration files:

*  Magento Open Source:
   *  `<your Magento 2 install dir>/vendor/magento/data-migration-tool/etc/opensource-to-opensource`: Configuration and scripts for migrating from Magento Open Source 1 to Magento Open Source 2

*  Adobe Commerce:
   *  `<your Magento 2 install dir>/vendor/magento/data-migration-tool/etc/opensource-to-commerce`: Configuration and scripts for migrating from Magento Open Source 1 to Adobe Commerce 2
   *  `<your Magento 2 install dir>/vendor/magento/data-migration-tool/etc/commerce-to-commerce`: Configuration and scripts for migrating from Adobe Commerce 1 to Adobe Commerce 2

The preceding directories contain subdirectories for each supported version.

## Configuring the migration

There are two ways to configure the [!DNL Data Migration Tool]:

*  Configure the [!DNL Data Migration Tool] in a separate module (recommended)
*  Change the [!DNL Data Migration Tool] configuration in the `<your Magento 2 install dir>/vendor/magento/data-migration-tool/etc/` directory.

To use source control to manage your migration configuration and use it for deployment, you must create a separate module.
If you plan to run the [!DNL Data Migration Tool] locally only, you can edit files in the `<your Magento 2 install dir>/vendor/magento/data-migration-tool/` directory directly.

### Configure migration in a separate module

Before you migrate any data, you must create a Magento 2 module.

1. Create a Magento 2 module.

   *  `<your Magento 2 install dir>/app/code/Vendor/Migration/composer.json`

   ```json
   {
       "name": "vendor/migration",
       "description": "Providing config for migration",
       "config": {
           "sort-packages": true
       },
       "require": {
           "magento/framework": "*",
           "magento/data-migration-tool": "*"
       },
       "type": "magento2-module",
       "autoload": {
           "files": [
               "registration.php"
           ],
           "psr-4": {
               "Vendor\\Migration\\": ""
           }
       },
       "version": "1.0.0"
   }
   ```

   *  `<your Magento 2 install dir>/app/code/Vendor/Migration/registration.php`

   ```php
   <?php

   \Magento\Framework\Component\ComponentRegistrar::register(
       \Magento\Framework\Component\ComponentRegistrar::MODULE,
       'Vendor_Migration',
       __DIR__
   );
   ```

   *  `<your Magento 2 install dir>/app/code/Vendor/Migration/etc/module.xml`

   ```xml
   <?xml version="1.0"?>

   <config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:noNamespaceSchemaLocation="urn:magento:framework:Module/etc/module.xsd">
       <module name="Vendor_Migration" setup_version="1.0.0">
           <sequence>
               <module name="Magento_DataMigrationTool"/>
           </sequence>
       </module>
   </config>
   ```

1. Copy the `config.xml.dist` configuration file from the appropriate directory of the [!DNL Data Migration Tool] (`<your Magento 2 install dir>/vendor/magento/data-migration-tool/etc/<migration edition>/<ce or version>`) into the `<your Magento 2 install dir>/app/code/Vendor/Migration/etc/<migration edition>/<ce or version>/config.xml` file.

   For example, if you migrate `Magento 1.9.3.6 Community Edition` to `Magento 2 Open Source`:

   ```bash
   cd <your Magento 2 install dir>
   ```

   ```bash
   cp vendor/magento/data-migration-tool/etc/opensource-to-opensource/1.9.3.6/config.xml.dist app/code/Vendor/Migration/etc/opensource-to-opensource/1.9.3.6/config.xml
   ```

1. In the `config.xml` file, you must set access details to M1 and M2 databases and encryption key.

1. If your M1 store has custom changes, you should map the rest of your configuration files to your Magento 1 store customizations. See [Work with configuration and mapping files](#migration-config).

### Configure migration in `vendor` folder

Before you migrate any data, you must create a `config.xml` configuration file from the provided sample.

To configure the [!DNL Data Migration Tool] for migration:

1. Log in to your application server as, or switch to, the [file system owner](../../installation/prerequisites/file-system/overview.md).

1. Change to the following directory:

   ```bash
   <your Magento 2 install dir>/vendor/magento/data-migration-tool/etc/<migration edition>/<ce or version>
   ```

1. Enter the following command to create a `config.xml` from the provided sample:

   ```bash
   cp config.xml.dist config.xml
   ```

1. Open `config.xml` in a text editor.

1. At a minimum, the config.xml file must contain access details to M1 and M2 databases and encryption keys.

   ```xml
   <source>
      <database host="127.0.0.1" name="magento1" user="root"/>
   </source>
   <destination>
      <database host="127.0.0.1" name="magento2" user="root"/>
   </destination>
   <options>
      <crypt_key />
   </options>
   ```

   The <crypt_key> tag must contain a value. You can find it inside the `<key>` tag, which is located in the app/etc/local.xml file on your Magento 1 instance.

   Optional parameters:

   *  Database user password: `password=<password>`
   *  Database custom port: `port=<port>`
   *  Table prefix: `<source_prefix>`, `<dest_prefix>`

   For example, if your database owner's username is `root` with password `pass` and you use the prefix `magento1` in your Magento 1 database, use the following in `config.xml`:

   ```xml
   <source>
      <database host="127.0.0.1" name="magento1" user="root" password="pass"/>
   </source>
   <destination>
      <database host="127.0.0.1" name="magento2" user="root" password="pass"/>
   </destination>
   <options>
      <source_prefix>magento1</source_prefix>
      <crypt_key>f3e25abe619dae2387df9fs594f01985</crypt_key>
   </options>
   ```

When finished, save your changes to `config.xml` and exit the text editor.

### Connect using the TLS protocol

You can also connect to a database using the TLS protocol (i.e., using public/private cryptographic keys). Add the following optional attributes to the `database` element:

*  `ssl_ca`
*  `ssl_cert`
*  `ssl_key`

For example:

```xml
<source>
    <database host="localhost" name="magento1" user="root" ssl_ca="/path/to/file" ssl_cert="/path/to/file" ssl_key="/path/to/file"/>
</source>
<destination>
    <database host="localhost" name="magento2" user="root" ssl_ca="/path/to/file" ssl_cert="/path/to/file" ssl_key="/path/to/file"/>
</destination>
```

## Work with configuration and mapping files

The [!DNL Data Migration Tool] uses *mapping files* to enable you to perform custom database mapping between your Magento 1 and Magento 2 databases, including:

*  Changing table names

*  Changing field names

*  Ignoring tables or fields

*  Adapt transferring data of a field to Magento 2 format

Mapping files for supported Magento versions are located in subdirectories of `<your Magento 2 install dir>/vendor/magento/data-migration-tool/etc`

To use the mapping files:

1. Copy them from `<your Magento 2 install dir>/vendor/magento/data-migration-tool/etc/<migration edition>/<ce or version>/` to `<your Magento 2 install dir>/app/code/Vendor/Migration/etc/<migration edition>/<ce or version>/` and remove the `.dist` extension.

1. Update the path to the newly copied file in the `<options>` node of `config.xml`. The updated path should be one of the following:

   1. Absolute file path, e. g. `/var/www/html/app/code/Vendor/Migration/etc/opensource-to-opensource/1.9.4.1/map.xml`
   1. magento/data-migration-tool module relative file path: `etc/opensource-to-opensource/1.9.4.1/map.xml`
   1. Magento root-relative file path: `app/code/Vendor/Migration/etc/opensource-to-opensource/1.9.4.1/map.xml`

The `<Magento 2 dir>/vendor/magento/data-migration-tool/etc` and `<Magento 2 dir>/vendor/magento/data-migration-tool/etc/<ce version>` directories contain the following configuration files:

Even though you are working with the `map.xml.dist` file most of the time, the following table discusses each mapping and other files.

| Mapping file name | Description |
| --- | --- |
| `class-map.xml.dist` | Dictionary of class mappings between Magento 1 and Magento 2 |
| `config.xml.dist` | Main configuration file that specifies the Magento 1 and Magento 2 database configurations, step configuration, and links to mapping files |
| *Adobe Commerce only*. `customer-attr-document-groups.xml.dist` | List of tables used in the custom customer attributes step. |
| *Adobe Commerce only*. `customer-attr-map.xml.dist` | Map file that is used in Custom Customer Attributes Step. |
| `deltalog.xml.dist` | Contains the list of tables required for database routines setup. |
| `eav-attribute-groups.xml.dist` | Contains list of attributes that are used in Eav Step. |
| `eav-document-groups.xml.dist` | Contains list of tables that are used in Eav Step. |
| `log-document-groups.xml.dist` | Contains list of tables that are used in Log Step. |
| `map-eav.xml.dist` | Map file that is used in EAV Step. |
| `map-log.xml.dist` | Log-mapping file. |
| *Adobe Commerce only*. `map-sales.xml.dist` | Map file that is used in SalesOrder Step. |
| `map.xml.dist` | Mapping file required for the map step. |
| `settings.xml.dist` | Setting migration configuration file that specifies rules required for migrating the `core_config_data` table. |
| `customer-attribute-groups.xml.dist` | Contains list of attributes that are used in Customer Attributes Step. |
| `customer-document-groups.xml.dist` | Contains list of tables that are used in Customer Attributes Step. |
| `map-customer.xml.dist` | Map file that is used in Customer Attributes Step. |
| `order-grids-document-groups.xml.dist` | Contains list of tables that are used in OrderGrids Step. |
| `map-document-groups.xml.dist` | Defines what fields are updated when duplications occur on data insert |
| `map-stores.xml.dist` | Map file that is used in Stores Step. |
| `map-tier-price.xml.dist` | Map file that is used in Tier Price Step. |
| *Adobe Commerce only*. `visual_merchandiser_map.xml.dist` | Map file that is used in VisualMerchandiser Step. |
| *Adobe Commerce only*. `visual_merchandiser_attribute_groups.xml.dist` | Contains list of attributes that are used in VisualMerchandiser Step. |
| *Adobe Commerce only*. `visual_merchandiser_document_groups.xml.dist` | Contains list of tables that are used in VisualMerchandiser Step. |

You can refer to [[!DNL Data Migration Tool] Technical Specification](technical-specification.md) for more details.
