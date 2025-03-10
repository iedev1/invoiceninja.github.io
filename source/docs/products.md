---
extends: _layouts.docs
section: content
---

# Products

## Creating Products

There several ways for a product to be created, including:

* Admin Portal > New Product
* Admin Portal > Settings > Import | Export > Import .CSV documents for batch product creation or imports.
* Manually entering new product information on a new line of an invoice or quote.  **Note** that when using this method, the product *quantity* used in the first invoice will be set as the *Default Quantity* for that product.
* As a developer you can create API calls to create, update, delete, export, or perform bulk actions on products, using API references from the Invoice Ninja [API Documentation](https://app.swaggerhub.com/apis/invoiceninja/invoiceninja/).  Templates for Integratomat and others have not yet been implemented.

Products can also be used to represent services rendered.  For example, you could have a product entry for service calls, with a price set to your hourly rate, and use the product quantity to represent the billable hours.

## Viewing Products

To view products from the Invoice Ninja admin portal, you must visit the Products module on the left hand navigation menu.  Alternatively, you can view and analyze your products list in an external application by exporting products as a .CSV file using the API, or the Export function under *Settings*.

### Overview

The *Overview* pane presents a very simple layout, with the product price in large text at the top, followed by the product description underneath.  

### Documents

The *Documents* pane provides the ability to upload documents and view documents you have linked to the product.  These uploaded files are *only* accessible through the admin portal.  A useful way to employ the document uploads feature, is by posting product signage, or documents with thorough product descriptions or technical details.  

**Note** that uploaded documents are saved in the "public/storage" directory in a folder structure using hashed folder names to match the product entry, so backup this directory along with your database to preserve your attached documents.

### Functions

There are a few functions available from the product view mode that provide shortcuts to manipulating the product you are viewing.

* **Edit** - The *Edit* button at the top right corner of the panel will allow you to edit the details of the product, such as the product name, description, price, and default quantity.  
* **New Invoice** - This button at the bottom of the panel will create a new invoice and take you to a *New Invoice* page, with the product you're viewing as a line item and the default quantity for that item already entered.
* **Clone** - This button will take you to a *New Product* screen, with the exactly same product details as the product you are viewing, allowing you to easily clone your product, and make any edits you need to before saving it as a new product.

## Editing a Product

There is only a few fields that apply to a product:

* **Product** - This is the name of the product itself, which will appear on invoices.
* **Description** - The product description, which will appear on invoices.  **Note** that PDF generation of invoices and quotes will process any HTML formatting you use here.  Furthermore, when *Enable Markdown* is turned on under *Settings* > *Account Management*, you will be able to enter markdown text into the product descriptions also, and it will appear formatted in your invoices, quotes, etc.
* **Price** - The standard price of your product.
* **Default Quantity** - The default quantity is used automatically when the product is added to an invoice or quote.

<x-next url=/docs/invoices>Invoices</x-next>