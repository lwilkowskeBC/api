# v3 Catalog API Documentation

_Welcome! Note that this API is in our partner release stage, which means we'll be iterating on feedback from partners in the short term to make sure all concerns are addressed and we reach our goal of creating the most powerful, easiest-to-utilize catalog API in commerce. Because of this, expect small changes and many additions to occur._

**Have suggestions, feedback or questions? Submit them as an issue here:** https://github.com/bigcommerce/api/issues

**Want to see what we have in development and help direct our roadmap? View our public API roadmap here:** https://trello.com/b/1Od4oCsl/bigcommerce-api-roadmap

## Access & Authentication

All BigCommerce stores have access to the v3 Catalog API.

The base URI: https://api.bigcommerce.com/stores/{store_hash}/v3/

To authenticate you'll need to use an OAuth client id and token, sent along with the following headers:
- Accept: application/json
- X-Auth-Client: {client_id}
- X-Auth-Token: {oauth_token}

The flow to register for a client id and retrieve a token is the same as the v2 API.
- To get your Client ID, you must complete [App Registration](https://developer.bigcommerce.com/api/registration).
- To get your OAuth token, you must complete [App Installation](https://developer.bigcommerce.com/api/callback).

On our short-term roadmap is the ability to more easily create OAuth credentials, from within the control panel, similarly to the way legacy v2 keys are created now. _Note that we'll be deprecating legacy keys from v2 and removing the ability to create within the CP in the future so grokking our OAuth flow now is not a wasted effort!_

Existing v2 client ids and tokens will also work with the v3 API. So if you've already integrated with v2 using our OAuth flow you should be golden!

## What's new?

- Variants
  - Every purchasable entity in the catalog is now a variant. When you create a product without any options, we automatically create a variant for you. This creates enhanced flows around inventory management, like the ability to solely use the variants endpoint to manage inventory levels.
- Options & Modifiers
  - There is now a clear separation of options that define variants and those that are modifiers of a variant. This enables us to simplify the creation and management of variant prices and modifier adjusters, removing the need to use complex rules in all but the most extreme cases.
  - Options and Modifiers can also be attached directly to the product, without the requirement to create an Option Set beforehand.
- Creating a Product w/ it's Variants in one API call
  - When creating an initial catalog you can send the core product and variant data in the same request, which helps you create more performant, managable codebases. We'll handle the Option and Option Value creation for you.
- Including variants in Product GETs
  - Following our goal of streamlining catalog management, you can now request to _?include=variants_ (along with other resources) to further eliminate API calls.
- Ready-made Catalog Tree
  - There's now an endpoint specifically for building out the catalog tree, which previously took a decent amount of work to construct.  
- Full access to modifer configuration values
  - Properties like number-only field limits and product list inventory adjustment settings are now available via this API. Over 20 properties previously unavailable to our developers.

## What's not here?

If your currently consuming our v2 API, there are some catalog endpoints and elements you'll notice are missing from this version. Some of the items are intentional and we're iterating on others, making sure they're done right.

- Intentional
  - Product -> Configurable Fields
    - Modifiers should now be used for any use case you'd use configurable fields for, as you can attach them to products as well and they cover a larger array of uses. 
  - Option Sets
    - In v3 you attach options directly to products, so option sets are not required and there is no endpoint in v3 to manage them. However, v3 will respect option sets that have been attached via v2 or the control panel.
- Iterating
  - Product -> Complex Rules
    - Keep in mind the majority of rule use cases can be solved through variant properties and modifiers adjusters already.
  - Product -> Reviews
  - Product -> Videos
  - Product -> Downloads
  - Product -> Google Search Mappings
    - May bring fields onto variant entity as properties / metadata instead
  - Product -> Open Graph and Accounting fields
    - May group into their own product objects to keep resource clean

## v2 Catalog API and Control Panel Interoperability

The v3 Catalog API is essentially our catalog's future state. This means that many concepts don't map visibly to their v2 and control panel relatives.

The good news here is we've built this API with v2 interoperability in mind, so you should be able to utilize both APIs simultaneously as you hopefully fully transition all catalog management to v3. The key areas to be aware of are:
- Option Sets
  - The Product resource on v3 has an option_set_id field that, if set, will prevent you from editing product options and modifiers directly. You will need to use v2 if you want to edit the Option Set or set the option_set_id field to null, which will remove the Option Set and allow you to attach options and modifiers directly.
  - In the Add / Edit Product section of our control panel any products created by v3 will have not have an Option Set applied, but merchants can still edit the Options. If the merchant edits chooses an Option Set any variants will be removed from the product.
- Product Rules
  - Any variant created in v3 with non-null core properties (price, weight, image, purchasablilty) will create a rule under the hood. Same with Modifier adjusters. These will show in v2 as Product Rules and any edits to them will shared across versions.

We're already working on refreshing the current add / edit product experience in our control panel to align with the concepts in v3.

### Product POST w/ Variants

When you include variants in your Product POST, we'll automatically create all the options and option values for you. If you don't pass the price and weight with the variants, the product price and weight will be used for the variants on the storefront.

Here's a sample POST to https://api.bigcommerce.com/stores/{store-hash}/v3/catalog/products
```javascript
{
    "name": "T-shirt",
    "type": "physical",
    "price": 10.25,
    "description": "<h4>Great T-shirt</h4>The best t-shirt ever.",
    "weight": 1.20,
    "categories": [
        18
    ],
    "variants": [
        {
            "sku": "SKU-R-SM",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Red"
                },
                {
                    "option_display_name": "Size",
                    "label": "Small"
                }
            ]
        },
        {
            "sku": "SKU-B-SM",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Blue"
                },
                {
                    "option_display_name": "Size",
                    "label": "Small"
                }
            ]
        },
        {
            "sku": "SKU-R-MD",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Red"
                },
                {
                    "option_display_name": "Size",
                    "label": "Medium"
                }
            ]
        },
        {
            "sku": "SKU-B-MD",
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Blue"
                },
                {
                    "option_display_name": "Size",
                    "label": "Medium"
                }
            ]
        },
        {
            "sku": "SKU-R-LG",
            "price": 10.50,
            "weight": 1.25,
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Red"
                },
                {
                    "option_display_name": "Size",
                    "label": "Large"
                }
            ]
        },
        {
            "sku": "SKU-B-LG",
            "price": 10.50,
            "weight": 1.25,
            "option_values": [
                {
                    "option_display_name": "Color",
                    "label": "Blue"
                },
                {
                    "option_display_name": "Size",
                    "label": "Large"
                }
            ]
        }
    ]
}
```

When you create a Product, we'll automatically return variants back in the response:
```javascript
{
    "data": {
        "id": 114,
        "name": "T-shirt",
        "type": 1,
        "sku": "",
        "description": "<h4>Great T-shirt</h4>The best t-shirt ever.",
        "weight": 1.2,
        "width": 0,
        "depth": 0,
        "height": 0,
        "price": 10.25,
        "cost_price": 0,
        "retail_price": 0,
        "sale_price": 0,
        "tax_class_id": 0,
        "product_tax_code": "",
        "calculated_price": 10.25,
        "categories": [
            18
        ],
        "brand_id": 0,
        "option_set_id": null,
        "inventory_level": 0,
        "inventory_warning_level": 0,
        "inventory_tracking": 0,
        "fixed_cost_shipping_price": 0,
        "is_free_shipping": false,
        "is_visible": true,
        "is_featured": false,
        "warranty": "",
        "bin_picking_number": "",
        "layout_file": "",
        "upc": "",
        "search_keywords": "",
        "availability": "available",
        "availability_description": "",
        "gift_wrapping_options": 0,
        "sort_order": 0,
        "condition": "New",
        "is_condition_shown": true,
        "order_quantity_minimum": 0,
        "order_quantity_maximum": 0,
        "page_title": "",
        "meta_keywords": [],
        "meta_description": "",
        "date_created": "2016-07-03T00:39:00+00:00",
        "date_modified": "2016-07-03T00:39:00+00:00",
        "view_count": 0,
        "preorder_release_date": null,
        "preorder_message": "",
        "is_preorder_only": false,
        "is_price_hidden": false,
        "price_hidden_label": "",
        "custom_url": {
            "url": "/t-shirt/",
            "is_customized": false
        },
        "variants": [
            {
                "id": 78,
                "product_id": 114,
                "sku": "SKU-R-SM",
                "sku_id": 127,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 98,
                        "option_id": 113
                    },
                    {
                        "id": 99,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 79,
                "product_id": 114,
                "sku": "SKU-B-SM",
                "sku_id": 128,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 100,
                        "option_id": 113
                    },
                    {
                        "id": 99,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 80,
                "product_id": 114,
                "sku": "SKU-R-MD",
                "sku_id": 129,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 98,
                        "option_id": 113
                    },
                    {
                        "id": 101,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 81,
                "product_id": 114,
                "sku": "SKU-B-MD",
                "sku_id": 130,
                "price": null,
                "weight": null,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 100,
                        "option_id": 113
                    },
                    {
                        "id": 101,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 82,
                "product_id": 114,
                "sku": "SKU-R-LG",
                "sku_id": 131,
                "price": 10.5,
                "weight": 1.25,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 98,
                        "option_id": 113
                    },
                    {
                        "id": 102,
                        "option_id": 114
                    }
                ]
            },
            {
                "id": 83,
                "product_id": 114,
                "sku": "SKU-B-LG",
                "sku_id": 132,
                "price": 10.5,
                "weight": 1.25,
                "purchasing_disabled": false,
                "purchasing_disabled_message": "",
                "image_file": null,
                "cost_price": 0,
                "upc": "",
                "inventory_level": 0,
                "inventory_warning_level": 0,
                "bin_picking_number": "",
                "option_values": [
                    {
                        "id": 100,
                        "option_id": 113
                    },
                    {
                        "id": 102,
                        "option_id": 114
                    }
                ]
            }
        ],
        "images": [],
        "custom_fields": [],
        "bulk_pricing_rules": []
    },
    "meta": {}
}
```

## Expanding Product Sub-resources on GET

You can include sub-resources on a product with a comma separated list by using *include={sub-resources}* as a query string. Valid expansions currently include `variants`, `images`, `custom_fields`, and `bulk_pricing_rules`. For instance, if you wanted variants and custom fields to also return in the product response, you'd GET: https://api.bigcommerce.com/stores/{store-hash}/v3/catalog/products?include=variants,custom_fields

## v3 Catalog API Reference

View the documentation generated from the Swagger file [here](https://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/v3-catalog.yaml).

### Endpoints

HTTP request | Description
------------- | -------------
**GET** /catalog/products | Returns a paginated collection of Products objects from the BigCommerce Catalog.
**POST** /catalog/products | Creates a Product in the BigCommerce Catalog
**DELETE** /catalog/products | Deletes one or more `Product` objects from the BigCommerce Catalog by an id or filter
**GET** /catalog/products/{product_id} | Returns a Product from the BigCommerce Catalog
**PUT** /catalog/products/{product_id} | Updates a Product in the BigCommerce Catalog
**DELETE** /catalog/products/{product_id} | Deletes a Product object from the BigCommerce Catalog
**GET** /catalog/products/{product_id}/variants | Returns a Variant object list from the BigCommerce Catalog.
**POST** /catalog/products/{product_id}/variants | Create a Variant object.
**GET** /catalog/products/{product_id}/variants/{variant_id} | Get a Variant object.
**PUT** /catalog/products/{product_id}/variants/{variant_id} | Update a Variant object.
**DELETE** /catalog/products/{product_id}/variants/{variant_id} | Delete a Variant
**POST** /catalog/products/{product_id}/variants/{variant_id}/image | Add an image to a variant that will show on the storefront when it's selected
**GET** /catalog/products/{product_id}/options | Get an array of Option objects.
**POST** /catalog/products/{product_id}/options | Create an Option.
**PUT** /catalog/products/{product_id}/options/{option_id} | Update a Product's Option based on the product_id and option_id.
**DELETE** /catalog/products/{product_id}/options/{option_id} | Delete a Product's Option based on the product_id and option_id.
**GET** /catalog/products/{product_id}/modifiers | Get an array of Modifier objects.
**POST** /catalog/products/{product_id}/modifiers | Create a Modifier.
**GET** /catalog/products/{product_id}/modifiers/{modifier_id} | Get a Product Modifier by modifier_id
**PUT** /catalog/products/{product_id}/modifiers/{modifier_id} | Update an Product's Modifier based on the product_id and modifier_id.
**DELETE** /catalog/products/{product_id}/modifiers/{modifier_id} | Delete a Product's Modifier based on the product_id and modifier_id.
**POST** /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}image | Add an image to a modifier value that will show on the storefront when it's selected
**DELETE** /catalog/products/{product_id}/modifiers/{modifier_id}/values/{value_id}/image | Delete the image applied to show when the modifier value is selected
**GET** /catalog/variants | Returns a Variant object list from the BigCommerce Catalog.
**GET** /catalog/categories | Returns a paginated categories collection from the BigCommerce Catalog
**POST** /catalog/categories | Creates a Category in the BigCommerce Catalog
**DELETE** /catalog/categories | Delete a product or products from the BigCommerce Catalog
**GET** /catalog/categories/{category_id} | Returns a Category from the BigCommerce Catalog
**PUT** /catalog/categories/{category_id} | Update a Category in the BigCommerce Catalog
**DELETE** /catalog/categories/{category_id} | Delete one or more Catggory objects from the BigCommerce catalog.
**POST** /catalog/categories/{category_id}/images | Creates an image on a category. Publically accessible URLs and files (form post) are valid parameters.
**DELETE** /catalog/categories/{category_id}/images | Delete a Category image the BigCommerce Catalog
**GET** /catalog/categories/tree | Returns the categories tree, a nested lineage of the categories with parent->child relationship. The Category objects returned are a simplified version of the category objects returned in the rest of this API.
**GET** /catalog/brands | Returns a paginated collection of Brand objects from the BigCommerce Catalog.
**POST** /catalog/brands | Create a Brand object.
**DELETE** /catalog/brands | Delete one or more Brand objects from the BigCommerce Catalog
**GET** /catalog/brands/{brand_id} | Gets Brand objects.
**PUT** /catalog/brands/{brand_id} | Updates a Brand in the BigCommerce Catalog
**DELETE** /catalog/brands/{brand_id} | Delete a Brand from the BigCommerce Catalog
**POST** /catalog/brands/{brand_id}/images | Creates an image on a Brand. Publically accessible URLs and files (form post) are valid parameters.
**DELETE** /catalog/brands/{brand_id}/images | Delete a Brand image the BigCommerce Catalog

### Models

##### Brand
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numerical ID of the brand, increments sequentially. | [optional] 
**name** | **string** | The name of the brand. Must be unique | [optional] 
**page_title** | **string** | The title shown in the browser while viewing the brand | [optional] 
**meta_keywords** | **string[]** | Comma separated list of meta keywords to include in the HTML | [optional] 
**meta_description** | **string** | A meta description to include | [optional] 
**search_keywords** | **string** | A comma separated list of keywords that can be used to locate this brand | [optional] 
**image_url** | **string** | A valid image | [optional] 


##### BulkPricingRule
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The ID of the bulk pricing rule. | [optional] 
**quantity_min** | **int** | The maximum inclusive quantity of a product to satisfy this rule. Must be greater than the min value, unless this field has a value of 0 (zero), in which case there will be no maximum bound for this rule. | [optional] 
**quantity_max** | **int** | The minimum inclusive quantity of a product to satisfy this rule. Must be greater than or equal to zero. | [optional] 
**type** | **string** | The type of adjument that is made, &#39;price&#39; - The adjument amount per product, &#39;percent&#39; - The adjustment as a percent of the original price, &#39;fixed&#39; - the adjusted absolute price of the product. | [optional] 
**amount** | **int** | The value of the adjusted by the bulk pricing rule. | [optional] 


##### Category
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numerical ID of the category, increments sequentially. | [optional] 
**parent_id** | **int** | The unique numerical ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog. | [optional] 
**name** | **string** | The name displayed for the category. Name is unique to the category&#39;s siblings. | [optional] 
**description** | **string** | The product description which can include HTML formatting. | [optional] 
**views** | **int** | Number of views the category has on the storefront | [optional] 
**sort** | **int** | Priority this category will be given when included in the menu and category pages. The lower the number, the closer to the top of the results the category will be. | [optional] 
**page_title** | **string** | Custom title for the category page, if not defined the category name will be used as the meta title. | [optional] 
**meta_keywords** | **string[]** | Custom meta keywords for the category page, if not defined the store default keywords will be used. Must post as an array like: [\&quot;awesome\&quot;,\&quot;sauce\&quot;] | [optional] 
**meta_description** | **string** | Custom meta description for the category page, if not defined the store default meta description will be used. | [optional] 
**layout_file** | **string** | The layout template file used to render this category. | [optional] 
**image_file** | **string** | URL of the image file shown for this category on the storefront. Images can be uploaded to /categories/{categoryId}/images. | [optional] 
**is_visible** | **bool** | Flag to determine if the product should be displayed to customers browsing the store or not. If true the category will be displayed, false the category will be hidden from view. | [optional] 
**search_keywords** | **string** | A comma separated list of keywords that can be used to locate the category when searching the store. | [optional] 
**default_product_sort** | **string** | Determines how the products are sorted on category page load. | [optional] 
**custom_url** | **string** | The custom url for the category on the store front. | [optional] 


##### CategoryNode
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numerical ID of the category, increments sequentially. | [optional] 
**parent_id** | **int** | The unique numerical ID of the category&#39;s parent. This field controls where the category sits in the tree of categories that organize the catalog. | [optional] 
**name** | **string** | The name displayed for the category. Name is unique to the category&#39;s siblings. | [optional] 
**is_visible** | **bool** | Flag to determine if the product should be displayed to customers browsing the store or not. If true the category will be displayed, false the category will be hidden from view. | [optional] 
**url** | **string** | The custom url for the category on the store front. | [optional] 
**children** | **array** | The list of children of the category. | [optional] 


##### CollectionMeta
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**total** | **int** | Total number of items return in the result set | [optional]
**count** | **int** | Total number of items in the collection | [optional] 
**per_page** | **int** | The amount of items returned in the collection per page, controlled by the limit parameter | [optional] 
**current_page** | **int** | The page you are currently on within the collection | [optional] 
**total_pages** | **int** | The total number of pages in the collection | [optional] 
**links** | **object** |  | [optional] 
**links.previous** | **string** | Link to the previous page returned in the response | [optional] 
**links.current** | **string** | Link to the current page returned in the response | [optional] 
**links.next** | **string** | Link to the next page returned in the response | [optional] 

##### Modifier
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numerical ID of the modifier, increments sequentially. | [optional] 
**product_id** | **int** | The unique numerical ID of the product that the option belongs to. | [optional] 
**name** | **string** | The name unique option name auto-generated from the display name, a timestamp, and the product id. | [optional] 
**display_name** | **string** | The name of the option shown on the store front. | [optional] 
**type** | **string** | Possible values: date, checkbox, file, text, multi_line_text, numbers_only_text,  radio_buttons, rectangles, dropdown, product_list, product_list_with_images, swatch | [optional] 
**config** | **object** | (See `OptionConfig` below) | [optional] 
**values** | **object** |  | [optional] 
**values.id** | **int** | The unique numerical ID of the value, increments sequentially. | [optional] 
**values.is_default** | **bool** | The flag for preselecting a value as the default on the store front. This field is not supported for swatch options/modifiers. | [optional] 
**values.label** | **string** | The text display identifying the value on the store front. | [optional] 
**values.sort_order** | **int** | The order in which the value will be displayed on the product page. | [optional] 
**values.value_data** | **object** | Extra data describing the value based on the type of option or modifier that the value is associated with swatch requires an array of colors with up to three hexidecimal color keys, product list requires a &#x60;product_id&#x60;, and checkbox requires boolean flag called &#x60;checked_value&#x60; to determine which value is considered to be the checked state. | [optional] 
**values.adjusters** | **object** | 
**values.adjusters.adjuster** | **string** | The type of adjuster as either &#x60;relative&#x60; or &#x60;percentage&#x60; for either the price or the weight of the variant when the modifier value is selected on the storefront. | [optional] 
**values.adjusters.adjuster_value** | **float** | The numeric amount the adjuster will change either the price or the weight of the variant when the modifier value is selected on the storefront. | [optional] 


##### Option
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numerical ID of the option, increments sequentially. | [optional] 
**product_id** | **int** | The unique numerical ID of the product that the option belongs to. | [optional] 
**name** | **string** | The name unique option name auto-generated from the display name, a timestamp, and the product id. | [optional] 
**display_name** | **string** | The name of the option shown on the store front. | [optional] 
**type** | **string** | Possible values: radio_buttons, rectangles, dropdown, product_list, product_list_with_images, swatch | [optional] 
**config** | **object** | (See `OptionConfig` below) | [optional] 
**values** | **object** |  | [optional] 
**values.id** | **int** | The unique numerical ID of the value, increments sequentially. | [optional] 
**values.is_default** | **bool** | The flag for preselecting a value as the default on the store front. This field is not supported for swatch options/modifiers. | [optional] 
**values.label** | **string** | The text display identifying the value on the store front. | [optional] 
**values.sort_order** | **int** | The order in which the value will be displayed on the product page. | [optional] 
**values.value_data** | **object** | Extra data describing the value based on the type of option or modifier that the value is associated with swatch requires an array of colors with up to three hexidecimal color keys, product list requires a &#x60;product_id&#x60;, and checkbox requires boolean flag called &#x60;checked_value&#x60; to determine which value is considered to be the checked state. | [optional] 


##### OptionConfig
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**default_value** | **string** | (date, text, multi_line_text, numbers_only_text) The default value shown on a date option as an ISO-8601 formatted string or text option as a string. | [optional] 
**checked_by_default** | **bool** | (checkbox) Flag for setting the check box to be checked by default. | [optional] 
**checkbox_label** | **string** | (checkbox) Label displayed for the checkbox option. | [optional] 
**date_limited** | **bool** | (date) Flag to limit the date allowed to be entered on a date option. | [optional] 
**date_limit_mode** | **string** | (date) The type of limit that is allowed to be entered on a date option (&#x60;earliest&#x60;, &#x60;range&#x60;, &#x60;latest&#x60;). | [optional] 
**date_earliest_value** | **int** | (date) The earliest date allowed to be entered on the date option as an ISO-8601 formatted string. | [optional] 
**date_latest_value** | **int** | (date) The latest date allowed to be entered on the date option as an ISO-8601 formatted string. | [optional] 
**file_types_mode** | **string** | (file) The kind of restriction on the type of files that can be uploaded to a file upload option.   &#x60;specific&#x60; - restricts upload to particular file types.   &#x60;all&#x60; - allows all files types. | [optional] 
**file_types_supported** | **string[]** | (file) The kind of files allowed to be uploaded if the &#x60;file_type_option&#x60; is set to &#x60;specific&#x60;.   &#x60;images&#x60; - Allows upload of image MIME types (&#x60;bmp&#x60;,&#x60;gif&#x60;,&#x60;jpg&#x60;,&#x60;jpeg&#x60;,&#x60;jpe&#x60;,&#x60;jif&#x60;,&#x60;jfif&#x60;,&#x60;jfi&#x60;,&#x60;png&#x60;,&#x60;wbmp&#x60;,&#x60;xbm&#x60;,&#x60;tiff&#x60;).   &#x60;documents&#x60; - Allows upload of document MIME types (&#x60;txt&#x60;,&#x60;pdf&#x60;,&#x60;rtf&#x60;,&#x60;doc&#x60;,&#x60;docx&#x60;,&#x60;xls&#x60;,&#x60;xlsx&#x60;,&#x60;accdb&#x60;,&#x60;mdb&#x60;,&#x60;one&#x60;,&#x60;pps&#x60;,&#x60;ppsx&#x60;,&#x60;ppt&#x60;,&#x60;pptx&#x60;,&#x60;pub&#x60;,&#x60;odt&#x60;,&#x60;ods&#x60;,&#x60;odp&#x60;,&#x60;odg&#x60;,&#x60;odf&#x60;).   &#x60;other&#x60; - Allows other file type defined in the &#x60;file_types_other&#x60; array. | [optional] 
**file_types_other** | **string[]** | (file) A list of other files types allowed with the file upload option. | [optional] 
**file_max_size** | **int** | (file) The maximum size of the file that can be used in the file upload option. | [optional] 
**text_characters_limited** | **bool** | (text, multi_line_text) Flag to validate the length of the text of a text or multi-line text input. | [optional] 
**text_min_length** | **int** | (text, multi_line_text) The minimum length allowed for a text or multi-line text option. | [optional] 
**text_max_length** | **int** | (text, multi_line_text) The maximum length allowed for a text or multi line text option. | [optional] 
**text_lines_limited** | **bool** | (multi_line_text) Flag to validate the maximum number of lines allowed on a multi-line text input. | [optional] 
**text_max_lines** | **int** | (multi_line_text) The maximum number of lines allowed on a multi-line text input. | [optional] 
**number_limited** | **bool** | (numbers_only_text) Flag to limit the value of a number option. | [optional] 
**number_limit_mode** | **string** | (numbers_only_text) The type of limit that is allowed to be entered on a number option (&#x60;lowest&#x60;, &#x60;highest&#x60;, &#x60;range&#x60;). | [optional] 
**number_lowest_value** | **float** | (numbers_only_text) The lowest allowed value for a number option if &#x60;limit_input&#x60; is true. | [optional] 
**number_highest_value** | **float** | (numbers_only_text) The highest allowed value for a number option if &#x60;limit_input&#x60; is true. | [optional] 
**number_integers_only** | **bool** | (numbers_only_text) Flag to limit the imput on a number option to only accept whole numbers. | [optional] 
**product_list_adjusts_inventory** | **bool** | (product_list, product_list_with_images) Flag for automatically adjusting inventory on a product included in the list. | [optional] 
**product_list_adjusts_pricing** | **bool** | (product_list, product_list_with_images) Flag to add the price of the optional product to the price of the main product | [optional] 
**product_list_shipping_calc** | **string** | (product_list, product_list_with_images)  How to factor the optional product&#39;s weight and dimensions (package) into the shipping quote (none - don&#39;t adjust, weight - use shipping weight only, package - use package, weight and dimensions) | [optional] 

##### OptionValue
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numerical ID of the value, increments sequentially. | [optional] 
**is_default** | **bool** | The flag for preselecting a value as the default on the store front. This field is not supported for swatch options/modifiers. | [optional] 
**label** | **string** | The text display identifying the value on the store front. | [optional] 
**sort_order** | **int** | The order in which the value will be displayed on the product page. | [optional] 
**value_data** | **object** | Extra data describing the value based on the type of option or modifier that the value is associated with swatch requires an array of colors with up to three hexidecimal color keys, product list requires a &#x60;product_id&#x60;, and checkbox requires boolean flag called &#x60;checked_value&#x60; to determine which value is considered to be the checked state. | [optional] 

##### Product
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** | The unique numerical ID of the product, increments sequentially. | [optional] 
**name** | **string** | The product name. | [optional] 
**type** | **int** | The product type: physical - a physical stock unit, digital - a digital download | [optional] 
**sku** | **string** | User defined product code/stock keeping unit (SKU). | [optional] 
**description** | **string** | The product description which can include HTML formatting. | [optional] 
**weight** | **double** | Weight of the product used which can be used when calculating shipping costs. | [optional] 
**width** | **double** | Width of the product used which can be used when calculating shipping costs. | [optional] 
**depth** | **double** | Depth of the product used which can be used when calculating shipping costs. | [optional] 
**height** | **double** | Height of the product used which can be used when calculating shipping costs. | [optional] 
**price** | **double** | The price of the product, the price should include or exclude tax based on the store settings. | [optional] 
**cost_price** | **double** | he cost price of the product, stored for reference only, it is not used or displayed anywhere on the store. | [optional] 
**retail_price** | **double** | The retail cost of the product, if entered the retail cost price will be shown on the product page. | [optional] 
**sale_price** | **double** | The sale price will be used instead of value in the price field when calculating the products cost if entered. | [optional] 
**tax_class_id** | **int** | The ID of the tax class applied to the product. NOTE: Value ignored if automatic tax is enabled. | [optional] 
**product_tax_code** | **string** | Accepts AvaTax system codes that identify products and services that fall into special sales tax categories. Allows merchants that subscribe to Avalara Premium to achieve increased accuracy in sales tax calculations. Stores without Avalara Premium will ignore the code when calculating sales tax. Do not pass more than one code. The codes are case-sensitive. Refer to the &#39;AvaTax System tax codes&#39; section of the following page for further information and the full list of codes: https://help.avalara.com/000_AvaTax_Calc/000AvaTaxCalc_User_Guide/040_Managing_Tax_Profiles/050_Tax_Codes/001_What_is_a_Tax_Code | [optional] 
**calculated_price** | **double** | The price of the product unless a sale_price is set. | [optional] 
**categories** | **int[]** | An array of IDs for the categories this product belongs to. When updating a product, if an array of categories is supplied all product categories will be overwritten. Does not accept more than 1,000 ID values. | [optional] 
**brand_id** | **int** | The id associated with the product&#39;s brand. | [optional] 
**inventory_level** | **int** | Current inventory level of the product. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect. | [optional] 
**inventory_warning_level** | **int** | Inventory Warning level for the product. When the products inventory level drops below the warning level the store owner will be informed. Simple inventory tracking must be enabled (See the inventory_tracking field) for this to take any effect. | [optional] 
**inventory_tracking** | **string** | The type of inventory tracking for the product: none - inventory levels will not be tracked. product - inventory levels will be tracked using the &#39;inventory_level&#39; and &#39;inventory_warning_level&#39; fields. variant - inventory levels will be tracked based on  variants which maintain their own warning levels and inventory levels. | [optional] 
**fixed_cost_shipping_price** | **int** | A fixed shipping cost for the product, if defined the value will be used during checkout instead of normal shipping cost calculation. | [optional] 
**is_free_shipping** | **bool** | Flag used to indicate if the product has free shipping or not. If true the shipping cost for the product will be zero. | [optional] 
**is_visible** | **bool** | Flag to determine if the product should be displayed to customers browsing the store or not. If true the product will be displayed, false the product will be hidden from view. | [optional] 
**is_featured** | **bool** | Flag to determine if the product should be included in &#39;featured products&#39; panel when viewing the store. | [optional] 
**warranty** | **string** | Warranty information displayed on the product page which can include HTML formatting. | [optional] 
**bin_picking_number** | **string** | The BIN picking number for the product. | [optional] 
**layout_file** | **string** | The layout template file used to render this product. | [optional] 
**upc** | **string** | The product UPC code which is used in feeds for shopping comparison sites and external channel integrations. | [optional] 
**search_keywords** | **string** | A comma separated list of keywords that can be used to locate the product when searching the store. | [optional] 
**availability** | **string** | Availability of the product. availability options: available - The product can be purchased in the store front. disabled - The product is listed in the store front but can not be purchased. preorder - The product is listed for pre-orders. | [optional] 
**availability_description** | **string** | Availability text displayed on the checkout page under the product title telling the customer how long it will normally take to ship this product, such as &#39;Usually ships in 24 hours&#39;. | [optional] 
**gift_wrapping_options** | **int[]** | A list of gift wrapping option ids, 0 (allow any gift wrapping options in the store), or -1 to disallow gift wrapping on the product. | [optional] 
**sort_order** | **int** | Priority this product will be given when included in product lists on category pages and search results. The lower the number, the closer to the top of the results the product will be. | [optional] 
**condition** | **string** | The product condition, will be shown on the product page if the value of the &#39;is_condition_shown&#39; field is true. Possible values: New, Used, Refurbished | [optional] 
**is_condition_shown** | **bool** | Flag used to determine if the product condition is shown to the customer on the product page. | [optional] 
**order_quantity_minimum** | **int** | The minimum quantity an order has to contain to be able to purchase this product. | [optional] 
**order_quantity_maximum** | **int** | The maximum quantity an order can contain when purchasing the product. | [optional] 
**page_title** | **string** | Custom title for the products page, if not defined the product name will be used as the meta title. | [optional] 
**meta_keywords** | **string[]** | Custom meta keywords for the product page, if not defined the store default keywords will be used. | [optional] 
**meta_description** | **string** | Custom meta description for the product page, if not defined the store default meta description will be used. | [optional] 
**date_created** | **string** | The date of which the product was created. | [optional] 
**date_modified** | **string** | The date of which the product was created. | [optional] 
**view_count** | **int** | The number of times the product has been viewed. | [optional] 
**preorder_release_date** | **string** | Pre-order release date. See availability field for details on setting a products availability to accept pre-orders. | [optional] 
**preorder_message** | **string** | Custom expected date message to display on the product page, if undefined the message defaults to the store wide setting. Can contain the %%DATE%% place holder which will be substituted for the release date | [optional] 
**is_preorder_only** | **bool** | If set to false, the product will not change its availability from preorder to available on the release date. Otherwise on the release date the products availability/status will change to available. | [optional] 
**is_price_hidden** | **bool** | False by default, which indicates that the price of this product should be shown on the product page. If set to &#39;true&#39;, the price is hidden. NOTE: To successfully set &#39;is_price_hidden&#39; to &#39;true&#39;, the &#39;availability&#39; value must be &#39;disabled&#39;. | [optional] 
**price_hidden_label** | **string** | By default, an empty string. If &#39;is_price_hidden&#39; is &#39;true&#39;, the value of &#39;price_hidden_label&#39; is displayed instead of the price. NOTE: To successfully set a non-empty string value for &#39;is_price_hidden&#39; to &#39;true&#39;, the &#39;availability&#39; value must be &#39;disabled&#39;. | [optional] 
**images** | **object** |  | [optional] 
**images.id** | **int** | The unique numerical ID of the image, increments sequentially | [optional] 
**images.product_id** | **int** | The unique numerical identifier for the product that the image is associated with. | [optional] 
**images.is_thumbnail** | **bool** | Flag for identifying if the image is used as the product&#39;s thumbnail. | [optional] 
**images.sort_order** | **int** | The order in which the image will be displayed on the product page. When updating if the image is given a lower priority, all image with a sort_order the same or greater than the images new sort_order value will have their sort_order reordered | [optional] 
**images.description** | **string** | The description for the image. | [optional]
**images.image_file** | **string** | The local path to the original image file uploaded to BigCommerce. | [optional] 
**images.url_zoom** | **string** | The zoom url for this image. | [optional] 
**images.url_standard** | **string** | Url The standard url for this image. | [optional] 
**images.url_thumbnail** | **string** | Url fThe thumbnail url for this image. | [optional] 
**images.url_tiny** | **string** | The tiny url for this image. | [optional] 
**custom_fields** | **object** |  | [optional] 
**custom_fields.id** | **int** | The unique numerical ID of the custom field, increments sequentially | [optional] 
**custom_fields.name** | **string** | The name of the field shown on the store front, orders, etc.. | [optional] 
**custom_fields.value** | **string** | The values or text of the field shown on the store front, orders, etc.. | [optional] 
**custom_url** | **object** |  | [optional] 
**custom_url.url** | **string** | Product url on the store front. | [optional] 
**custom_url.is_customized** | **bool** | Returns true if the url has been changed from it&#39;s default state (the auto-assigned url BigCommerce provides) | [optional] 
**bulk_pricing_rules** | **object** |  | [optional] 
**bulk_pricing_rules.id** | **int** | The ID of the bulk pricing rule. | [optional] 
**bulk_pricing_rules.quantity_min** | **int** | The maximum inclusive quantity of a product to satisfy this rule. Must be greater than the min value, unless this field has a value of 0 (zero), in which case there will be no maximum bound for this rule. | [optional] 
**bulk_pricing_rules.quantity_max** | **int** | The minimum inclusive quantity of a product to satisfy this rule. Must be greater than or equal to zero. | [optional] 
**bulk_pricing_rules.type** | **string** | The type of adjument that is made, &#39;price&#39; - The adjument amount per product, &#39;percent&#39; - The adjustment as a percent of the original price, &#39;fixed&#39; - the adjusted absolute price of the product. | [optional] 
**bulk_pricing_rules.amount** | **int** | The value of the adjusted by the bulk pricing rule. | [optional] 
**variants** | **object**  | (see below) | [optional] 

##### Variant
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** |  | [optional] 
**product_id** | **int** |  | [optional] 
**sku** | **string** |  | [optional] 
**sku_id** | **int** | read-only reference to V2 sku id, null if it&#39;s a base variant | [optional] 
**cost_price** | **string** | The cost price of the variant. | [optional] 
**price** | **string** | This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s price field) will be used as the base price. | [optional] 
**weight** | **string** | This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. | [optional] 
**purchasing_disabled** | **bool** | If true, this variant will not be purchasable on the storefront | [optional] 
**purchasing_disabled_message** | **string** | If purchasing_disabled is true, this message should show on the storefront when the variant is selected | [optional] 
**image_file** | **string** | The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. | [optional] 
**upc** | **string** | The UPC code which is used in feeds for shopping comparison sites and external channel integrations. | [optional] 
**inventory_level** | **int** | Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant | [optional] 
**inventory_warning_level** | **int** | When the variant hits this inventory level it is considered low stock | [optional] 
**bin_picking_number** | **string** | Identifies where in a warehouse the variant is located. | [optional] 
**option_values** | **object** | Array of option and option values ids that make up this variant. Will be empty if the variant is the product&#39;s base variant. | [optional] 
**option_values.id** | **int** |  | [optional] 
**option_values.option_id** | **int** |  | [optional] 


##### VariantPost
------

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**id** | **int** |  | [optional] 
**product_id** | **int** |  | [optional] 
**sku** | **string** |  | [optional] 
**sku_id** | **int** | read-only reference to V2 sku id, null if it&#39;s a base variant | [optional] 
**price** | **string** | This variant&#39;s base price on the storefront. If this value is null, the product&#39;s default price (set in the Product resource&#39;s price field) will be used as the base price. | [optional] 
**weight** | **string** | This variant&#39;s base weight on the storefront. If this value is null, the product&#39;s default weight (set in the Product resource&#39;s weight field) will be used as the base weight. | [optional] 
**purchasing_disabled** | **bool** | If true, this variant will not be purchasable on the storefront | [optional] 
**purchasing_disabled_message** | **string** | If purchasing_disabled is true, this message should show on the storefront when the variant is selected | [optional] 
**image_file** | **string** | The image that will be displayed when this variant is selected on the storefront. When updating a SKU image, send the publicly accessible URL. Supported image formats are JPEG, PNG, and GIF. Generic product images not specific to the variant should be stored on the product. | [optional] 
**cost_price** | **string** | The variant&#39;s cost price | [optional] 
**upc** | **string** | The UPC code which is used in feeds for shopping comparison sites and external channel integrations | [optional] 
**inventory_level** | **int** | Inventory level for the variant, which is used when the product&#39;s inventory_tracking is set to variant | [optional] 
**inventory_warning_level** | **int** | When the variant hits this inventory level it is considered low stock | [optional] 
**bin_picking_number** | **string** | Identifies where in a warehouse the variant is located. | [optional] 
**option_values** | **object** |  | [optional] 
**option_values.option_display_name** | **string** | The name of the option to be created on POST | [optional] 
**option_values.label** | **string** | The label of the option value to be created on POST | [optional] 



