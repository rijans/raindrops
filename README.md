(Work in Progress)

## CRUD Generator

Our model will have a array property called `$fields` which will hold all the information we need 
to generate forms/list pages/filter options etc. later on.

`$fields` array will contain the model's database fields as key and another configuration array as value.

```php

protected $fields = [

	'field_name' => [
		'label' => 'Field Label',
		'type' => 'input type'
		/* other configurations goes here */
	]

]


```

### Options

- `label` (required)

	label which will be used to create label tag of the form element. also for creating table header
	in list page.

- `type` (required)

	Form input type for a specific field. Currently implemented types:

	- basic html input types : `text`, `email`, `password`, `file`, `textarea`
	- rich text editor: `editor`
	- select dropdown: `select`
	- populating dropdown from another database table(foreign key): `select_db`
	- datepicker: `date`
	- timepicker: `time`

These two fields are *required* to generate a generic form input element. Below options can be used for further customization
of the field.

- `form`

	Set this to `false` if you don't want to create a form input field for this particular field.

- `index` 

	Add a `index` key and set it to `true` if you want this field to be shown on the list table. 

- `required`

	This will be `true` if this field is required.The html form input element will have a `required` attribute for basic client side validations. `TODO:` later we need to bring in from `validations` option so that we don't need to specify required seperately.

- `validations` 

	Validation rules for this field.The format is exactly like laravel's default validation rules format with only one minor difference. When using `unique` rule, add a `{id}` add the end of the rule to ignore this model's ID while updating
	e.g. `unique:<table_name>,<column_name>{id}`

- `classes` (TODO)

	If you want to use any custom classes for any particular field, you can add a `classes` option. Pass a comma seperated string
	of class names.

- `attributes` (TODO)

	Any extra attributes to generated for the input field.This can be particularly helpful to customize the input with default html attributes, e.g. specifying `min` & `max` attribute for a `number` field. just pass an array with the attributes name/value as key/value pair.

- `show`

	This option is to determine how the field's data should be displayed. This is used both for the list page table and for the details page of resource. 

	#### currently supported options

	- `exact` show the value exactly as databae without any modifications.
	- `string` will pass the value through ucfirst. helpful to be used with name type fields.
	- `details_link` as a link to the details page of the resource.
	- `url` just wrap the value inside an anchor tag.
	- `tel` anchor tag for phone numbers, should be used to make phone numbers clickable
		on mobile devices.
	- `mailto` mailto: links, for email address.
	- `enum` if the field type is `select` this option will get the actual text from the `options` array.
	- `img` to wrap the value around an image tag. to be used with image url fields.
	- `doc` if the field is a url for any downloadable resource, this will provide option to download it.
	- `time`
	- `datetime`
	- `relation` when the field is actually a foreign key for another table, you can use the relation to show any column from that related model. add an array as the value of the `relation` option. first element is the *name of the relation* and the second option is the *column name* from that related model.

- `filter`

	What type of input element is to be generated for filtering this field. 

	#### Currently suported

	- `enum` dropdown with options populated from `options` element
	- `date` for any type of date fields. it will generate a `field_name-from` and `field_name-to` date picker field.`TODO:` need a to implement a daterange picker plugin for this.
	- `boolean` this will generate a checkbox. to be used with any flag type fields.


The following options are required based on the value of `type`

- `options`

	This option is required when you set `select` or `select_db` as the input type. For `select`, you need to specify dropdown's value's as an array.

	e.g.

	```php
	'type' => 'select',
	'options' => [
		'option_1' => 'Option One'
	]

	```

	this will generate an option tag for the select tag: 

	```html
	<option value="option_1">Option One</option>
	```

	for `select_db`, `options` will be a simple array. first element is the name of the db field, which will be used as the option's value.(generally the `id` column of the table) and second element is the column with option's value. if you pass more than two column names, all those will be concatenated with a space and displayed as option text.

- `table`

	The name of the table to be used with `select_db` option.


### Example

```php
    /**
     * Fields
     */
    protected $fields = [

        'public_id' => [
            'label' => 'Device ID',
            'form' => false,
            'index' => true,
            'show' => 'exact'
        ],

        'serial_number' => [
            'label' => 'Serial Number',
            'type' => 'text',
            'validations' => 'unique:devices,serial_number{id}',
            'index' => true,
            'unique' => true,
            'filter' => 'string'
        ],

        'model_id' => [
            'label' => 'Model',
            'type' => 'select_db',
            'table' => 'device_models',
            'options' => ['id', 'model'],
            'index' => true,
            'show' => ['model', 'model_name']
        ],

        'notes' => [
            'label' => 'Notes',
            'type' => 'editor'
        ],

        'category' => [
            'label' => 'Category',
            'type' => 'select',
            'form' => false,
            'options' => [
                'individual' => 'Individual',
                'company' => 'Company'
            ],
            'classes' => [],
            'required' => true,
            'attributes' =>[],
            'validations' => 'required',
        ],


```

## Model Configuration

There's some additional properties we need to specify in our model.

- `$baseUrl`

	The route for the model. This will be used to generate create/view/edit/delete url later.

- `$entityName`

	Name of the model's entity in singular form. Used for generating titles for various pages. e.g. **Add New Product**, **View All Products**

- `$entityNamePlural`

	By default, the `$entityName` will be passed through laravel's `str_plural` function to generate plural form of the entity name. If you want to override this and set custom plural form, you can use this property.

- `$actions`

	When generating rows on a list page, by default, *view*, *edit*, *delete* these action links will be generated.But if you want to add extra action links for custom routes, you can add them in `$actions` array. e.g. you want to add *sale* link for all your products in product list page, you can add:

	```php
		protected $actions = [
	        'Sale' => '{url}/{id}/sale'
	    ];
	```

	`{url}` will be replaced with the route and `{id}` will be replaced for that specific resource's id

- `$paths`

	For the fields which holds path for any type of assets e.g. images, documents etc. you can add their paths in a `$paths` array. This will be handy when uploading/retrieving assets later.

	```php
		protected $paths = [
	        'client_photo' => 'uploads/client/client_photo'
	    ];
	```