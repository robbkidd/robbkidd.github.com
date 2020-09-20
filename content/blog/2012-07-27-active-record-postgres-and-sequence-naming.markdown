---
layout: post
title: "Active Record, PostgreSQL and Sequence Naming"
date: 2012-07-27 16:33
comments: true
categories: [Ruby, Rails, PostgreSQL]
---

If you use PostgreSQL to back your Active Record models, you should
check the current names for your tables and their sequences. Prior to
Active Record 3.2.7, renaming a table did not rename the associated
sequence for the table's primary key.

A demonstration may be in order.

<!-- more -->

```
robb@neoldian ~/code/pg_seq » rails generate model Product name:string description:text
      invoke  active_record
      create    db/migrate/20120727210734_create_products.rb
      create    app/models/product.rb
      invoke    test_unit
      create      test/unit/product_test.rb
      create      test/fixtures/products.yml

robb@neoldian ~/code/pg_seq » rake db:migrate
==  CreateProducts: migrating =================================================
-- create_table(:products)
NOTICE:  CREATE TABLE will create implicit sequence "products_id_seq" for serial column "products.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "products_pkey" for table "products"
   -> 0.0194s
==  CreateProducts: migrated (0.0196s) ========================================
```

Note that on line 11 the `CREATE TABLE` created an implicit sequence
`products_id_seq` for serial column `products.id`.

Let's rename the table.

```bash
robb@neoldian ~/code/pg_seq » rails generate migration RenameProductsToWidgets
      invoke  active_record
      create    db/migrate/20120727211235_rename_products_to_widgets.rb
```

```ruby db/migrate/20120727211235_rename_products_to_widgets.rb
class RenameProductsToWidgets < ActiveRecord::Migration
  def up
    rename_table :products, :widgets
  end

  def down
    rename_table :widgets, :products
  end
end
```

```bash
robb@neoldian ~/code/pg_seq » rake db:migrate
==  RenameProductsToWidgets: migrating ========================================
-- rename_table(:products, :widgets)
   -> 0.0019s
==  RenameProductsToWidgets: migrated (0.0020s) ===============================
```

What do you expect the sequence for the `id` column in the `widgets`
table to be named at this point? `widgets_id_seq`?

```sh
                                      Table "public.widgets"
   Column    |            Type             |                       Modifiers
-------------+-----------------------------+-------------------------------------------------------
 id          | integer                     | not null default nextval('products_id_seq'::regclass)
 name        | character varying(255)      |
 description | text                        |
 created_at  | timestamp without time zone | not null
 updated_at  | timestamp without time zone | not null
Indexes:
    "products_pkey" PRIMARY KEY, btree (id)
```

The who the what?

On line 4, the next value for the `widgets.id` column is coming
from a sequence named `products_id_seq`. This is not catastrophic
for small code bases, but imagine coming across this database
months or years from now. "Why's the widget table have a sequence
named 'products'?"

The good news is that
[AR v3.2.7 includes a fix for this](https://github.com/rails/rails/pull/7031).
When you rename a table, if the name of the sequence for the table's
primary key matches the AR default "tablename_columnname_seq", then
the sequence will be renamed as well. (I suppose the next order of
business is to get that index renamed, too.)

So you might want to go take a look at your PostgreSQL tables. Their sequences
probably have names that do not make any sense now.

You can use this gist in your Rails console as a quick way to check.

{% gist 3190645 %}
