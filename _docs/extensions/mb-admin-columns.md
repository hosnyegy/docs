---
title: MB Admin Columns
---

{% include installation.html %}

## Usage

**Make sure you know how to [register meta boxes](/registering-meta-boxes/) and [define fields](/field-settings/) before continuing!**

To show the admin column for a custom field, simply add `admin_columns` parameter for that field. The parameter can be in one of 3 following formats (from the simple to advanced configuration):

### 1. Simply display the admin column

```php
'admin_columns' => true,
```

This line tells the MB Admin Columns extension to display the meta value of the custom field in a custom column. This method uses the default configuration below:

- The column title is field name
- The column is added to the end of the table

### 2. Specify the column position

You can set the column in a specific position by one of the following rules:

```php
'admin_columns' => 'before title',
'admin_columns' => 'after title',
'admin_columns' => 'replace title',
```

The format is `'admin_columns' => 'type column'` where:

Param|Description
---|---
`type`|Must be `before`, `after` or `replace`. Specify the position of the custom column.
|`before`: Insert the column before an existing column
|`after`: Insert the column after an existing column
|`replace`: Replace an existing column by the new one
`column`|The target existing column

Using this configuration, you are able to insert the custom column in any position you want. Note that the column title is the field name by default.

### 3. Advanced configuration

To add more rules for the custom column, you can declare `admin_columns` parameter as an array which accepts the following keys:

```php
'admin_columns' => array(
    'position'   => 'after title',
    'title'      => __( 'Price', 'textdomain' ),
    'before'     => '$',
    'after'      => ' USD',
    'sort'       => true,
    'searchable' => true,
    'filterable' => false,
),
```

The meaning of keys are described below:

Key|Description
---|---
`position`|Specify where to insert the new column. Accept rule in the format `'type column'`. It's exactly the same as described in the #2 method above.
`title`|Column title. Optional. Default is the field name.
`before`|Custom HTML outputted before the column content. Optional.
`after`|Custom HTML outputted after the column content. Optional.
`sort`|Whether to sort the column by meta value? Optional. Default is `false`.
`searchable`|Allow to search posts by meta value. Optional. Default is `false`.
`filterable`|Allow to filter posts by custom taxonomy. Applied only if the field is `taxonomy` field. Default is `false`.

Note that the `sort` parameter is used to sort displayed posts by altering the WordPress query. It **works only with non-clonable and non-multiple fields**.

## Example

Assuming we have a custom post type 'Book' registered with the following code:

```php
add_action( 'init', 'prefix_register_book_post_type' );
function prefix_register_book_post_type() {
    $labels = array(
        'name'          => _x( 'Books', 'Post Type General Name', 'textdomain' ),
        'singular_name' => _x( 'Book', 'Post Type Singular Name', 'textdomain' ),
        'menu_name'     => __( 'Book', 'textdomain' ),
    );
    $args   = array(
        'labels'     => $labels,
        'supports'   => array( 'title', 'editor', ),
        'public'     => true,
        'taxonomies' => array( 'category' ),
        'menu_icon'  => 'dashicons-book-alt',
    );
    register_post_type( 'book', $args );
}
```

Here is the code to register a meta box, custom fields with additional parameters for admin columns:

```php
add_filter( 'rwmb_meta_boxes', 'prefix_book_meta_boxes' );
function prefix_book_meta_boxes( $meta_boxes ) {
    $meta_boxes[] = array(
        'title'      => __( 'Book Info', 'textdomain' ),
        'post_types' => 'book',
        'fields'     => array(
            array(
                'name'          => __( 'Cover', 'textdomain' ),
                'id'            => 'cover',
                'type'          => 'image_advanced',
                'admin_columns' => 'before title', // Show this column before 'Title' column
            ),
            array(
                'name'          => __( 'Author', 'textdomain' ),
                'id'            => 'book_author',
                'type'          => 'text',
                'admin_columns' => 'after title', // Show this column after 'Title' column
            ),
            array(
                'name'          => __( 'Category', 'textdomain' ),
                'id'            => 'category',
                'type'          => 'taxonomy',
                'taxonomy'      => 'category',
                'admin_columns' => array(
                    'position' => 'replace categories', // Replace default 'Categories' column
                    'title'    => 'Genre',              // Custom title
                ),
            ),
            array(
                'name'          => __( 'Publisher', 'textdomain' ),
                'id'            => 'publisher',
                'type'          => 'text',
                'admin_columns' => 'replace date', // Replace 'Date' column
            ),
            array(
                'name'          => __( 'Pages', 'textdomain' ),
                'id'            => 'pages',
                'type'          => 'number',
                'admin_columns' => true, // Just show this column
            ),
            array(
                'name'          => __( 'Price', 'textdomain' ),
                'id'            => 'price',
                'type'          => 'number',
                'admin_columns' => array(
                    'before' => '$',    // Show custom HTML before and after column value
                    'after'  => ' USD',
                    'sort'   => true,   // Sort this column
                ),
            ),
        ),
    );
    return $meta_boxes;
}
```

Here is the result:

![meta box admin columns](https://metabox.io/wp-content/uploads/2016/03/admin-columns-screenshot.png)

## Create custom admin columns without custom fields

By default, the extension is made to work with custom fields created by the Meta Box plugin. However, there are situations where you want to make it work with non-custom fields by Meta Box. And that's totally possible. Follow the steps below (note that it requires some coding):

**Step 1: Create a custom class for custom columns**

Create a new file `custom.php` (you can name it anything you want) in your plugin / theme folder with the following content:

```php
class Prefix_Custom_Admin_Columns extends MB_Admin_Columns_Post {
    public function columns( $columns ) {
        $columns  = parent::columns( $columns );
        $position = '';
        $target   = '';
        $this->add( $columns, 'column_id', 'Column Title', $position, $target );
        // Add more if you want
        return $columns;
    }
    public function show( $column, $post_id ) {
        switch ( $column ) {
            case 'column_id':
                echo 'Column content';
                break;
            // More columns
        }
    }
}
```

Note that the custom column(s) is added via the call `$this->add( $columns, 'column_id', 'Column Title', $position, $target );`. You can repeat that code to add as many column as you want.

In the `show` function, you must add code to display content of these columns.

**Step 2: Create an instance of the class**

In your plugin main file or `functions.php` of your theme, add the following code to create an instance of the class above:

```php
add_action( 'admin_init', 'prefix_add_custom_columns', 20 );
function prefix_add_custom_columns() {
    require_once 'custom.php';
    new Prefix_Custom_Admin_Columns( 'post', array() );
}
```

Note that the call `new Prefix_Custom_Admin_Columns( 'post', array() );` tells WordPress to add custom column(s) to `post`. You can change this to another post type if you want.