---
layout: post
title:  "How to extend product models (part one)"
date:   2017-03-31 10:53:00 +0100
categories: product extend
---

This blog post is the first of a planned series about extending Cart product models.

Extending models is nothing special and can be applied to other extensions and extbase models as well.
I will create a further extension (CartExtendedProduct) in parallel to writing this post and, in addition, provide a Link
to the repository at the end of this article.

I won't use the extension builder, so we will have to create some directories and files on our own. In the repository,
I will add a composer.json file and some other files, but I will ignore them in my post because they don't have an
effect on the function.

**The addition of a manufacturer field seems to be a useful target for this extension.** To make it as simple as
possible, I will limit the type of the new field to a string.
 
## 1. File structure

```
cart_extended_product
│
└─── Classes 
│    │
│    └─── Domain 
│         │
│         └─── Model
│              │
│              └─── Product
│                   └─── Product.php
│
└─── Configuration 
│    │
│    └─── TCA 
│    │    │
│    │    └─── Overrides
|    |         └─── tx_cart_domain_model_product_product.php
│    │
│    └─── TypoScript
│         └─── setup.txt
│
└─── Resources 
│    │
│    └─── Private 
│         │
│         └─── Language
|              └─── de.locallang_db.xlf
|              └─── locallang_db.xlf
│
└─── Tests 
│    │
│    └─── Unit 
│         │
│         └─── Domain 
│              │
│              └─── Model
│                   │
│                   └─── Product
│                        └─── ProductTest.php
│
└─── ext_emconf.php 
└─── ext_tables.sql   
```

After creating this structure, start to fill the files with some code.

## 2. Extend the database, TCA and model

### 2.1. Extend the database

Add the new database field '*manufacturer*'.

**`ext_tables.sql`**
```php
#
# Table structure for table 'tx_cart_domain_model_product_product'
#
CREATE TABLE tx_cart_domain_model_product_product (
    manufacturer varchar(255) DEFAULT '' NOT NULL,
);
```

### 2.2. Extend the TCA

In '*TCA/Overrides*', add the configuration file for the new field. It must have the same filename as the configuration it extends.

**`Configuration/TCA/Overrides/tx_cart_domain_model_product_product.php`**

```php
$temporaryColumns = [
    'manufacturer' => [
        'exclude' => 1,
        'label' => $_LLL . ':tx_cart_domain_model_product_product.manufacturer',
        'config' => [
            'type' => 'input',
            'size' => 30,
            'eval' => 'trim'
        ],
    ],
];

\TYPO3\CMS\Core\Utility\ExtensionManagementUtility::addTCAcolumns(
    'tx_cart_domain_model_product_product',
    $temporaryColumns
);
\TYPO3\CMS\Core\Utility\ExtensionManagementUtility::addToAllTCAtypes(
    'tx_cart_domain_model_product_product',
    'manufacturer',
    '',
    'after:title'
);
```

This is quite simple. I will skip the content of the translation files here.

After updating the the database through the InstallTool and clearing the cache, the new field should be available in the backend forms next to the title.

{% include image.html name="CartExtendedProduct_TCA.png" alt="edit the extended product in backend" title="edit the extended product in backend"  %}


### 2.3. Extend the product model

Next, the getter and setter functions in the product model has to be provided by this extension.

**`Classes/Domain/Model/Product/Product.php`**
```php
namespace Extcode\CartExtendedProduct\Domain\Model\Product;

class Product extends \Extcode\Cart\Domain\Model\Product\Product
{
    protected $manufacturer = '';

    public function getManufacturer()
    {
        return $this->manufacturer;
    }

    public function setManufacturer($manufacturer)
    {
        $this->manufacturer = $manufacturer;
    }
}
```

The new \Extcode\CartExtendedProduct\Domain\Model\Product extends the \Extcode\Cart\Domain\Model\Product\Product.
Please note that extra **\Product** in the namespace because in the second part of this article, I will extend other product models too.
This is also represented by grouping the models in folders.

## 3. Extbase should use the new product model

### 3.1. TypoScript configuration

The most important thing is to tell extbase to use the new product model.

```
config.tx_extbase {
    persistence {
        classes {
            Extcode\CartExtendedProduct\Domain\Model\Product\Product {
                mapping {
                    tableName = tx_cart_domain_model_product_product
                    recordType =
                }
            }
        }
    }
    objects {
        Extcode\Cart\Domain\Model\Product\Product.className   = Extcode\CartExtendedProduct\Domain\Model\Product\Product
    }
}
```

Because the extended product model do not have it's own database table, the persistence array tells extbase to use the extended database table.
The objects array tells extbase to use the *Extcode\CartExtendedProduct\Domain\Model\Product\Product* whenever an instance
of *Extcode\Cart\Domain\Model\Product\Product* is used. That's why the new getter function can be used in the template files.

### 3.2. Templating

After clearing the cache, the new getter appears in the Fluid template.
I added the *{product.manufacturer}* after the product title in my own project template file.

{% include image.html name="CartExtendedProduct_ManufacturerFrontend.png" alt="edit the extended product in backend" title="edit the extended product in backend"  %}

## 4. Limitations

Because of the configuration in TypoScript, it is possible to extend the product model only once.
This shouldn't be a problem as long as no third party extension extends the models.

## 5. Conclusion

In most cases, it is very easy to extend the product model. You can extend other models as well.

## 6. Links

* [Repository v0.1.0](https://github.com/extcode/cart_extended_product/releases/tag/v0.1.0)

## 7. Outlook for the next post

Sometimes, the product's attribute must available in the order, so in the next post, I will show you how to transfer a new product attribute to the order item.
