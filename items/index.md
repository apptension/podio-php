---
layout: default
active: items
---
# Working with Podio items

<span class="note">If you haven't read about [Podio* objects]({{site.baseurl}}/objects) yet, do so first.</span>

Apps and app items form the core of Podio and for that reason podio-php tries to make it as easy as possible to read and manipulate app items.

## Individual items
### Get item (basic)
### Item fields
#### Get field
#### Add field
#### Remove field
#### Change field value
### Create item
### Update item

## Item collections
One of the most common operations is getting a collection of items from an app, potentially with a filter applied. For this you can use [PodioItem::filter()](https://developers.podio.com/doc/items/filter-items-4496747). It returns a collection with two additional properties: filtered (total amount of items with the filter applied) and total (total amount of items in the app). You can iterate over this collection as normal.

{% highlight php startinline %}
$collection = PodioItem::filter(123); // Get items from app with app_id=123

print "The collection contains ".count($collection)." items";
print "There are ".$collection->total." items in the app in total";
print "There are ".$collection->filtered." items with the current filter applied";

// Output the title of each item
foreach ($collection as $item) {
  print $item->title;
}
{% endhighlight %}

### Sorting items
You can sort items by various properties. [See a full list in the API reference](https://developers.podio.com/doc/filters).
{% highlight php startinline %}
// Sort by last edit date for the items, descending
$collection = PodioItem::filter(123, array(
  'sort_by' => 'last_edit_on',
  'sort_desc' => true
));
{% endhighlight %}

### Filters
You can filter on most fields. Take a look at the [API reference for a full list of filter options](https://developers.podio.com/doc/filters). When filtering on app fields use the `field_id` or `external_id` as the key for your filter. Some examples below:

{% highlight php startinline %}
// Category: Only items with "FooBar" in category field value
$category_field_id = 1;
$collection = PodioItem::filter(123, array(
  'filters' => array(
    $category_field_id => array("FooBar")
  ),
));
{% endhighlight %}

{% highlight php startinline %}
// Number: Only items within a certain range
// Same concept for calculation, progress, duration & money fields
$number_field_id = 2;
$collection = PodioItem::filter(123, array(
  'filters' => array(
    $number_field_id => array("from" => 100, "to" => 200)
  ),
));
{% endhighlight %}

{% highlight php startinline %}
// App reference: Only items that has a specific reference
$app_reference_field_id = 3;

// Item id to filter against
$filter_target_item_id = 123456789;

$collection = PodioItem::filter(123, array(
  'filters' => array(
    $app_reference_field_id => array($filter_target_item_id)
  ),
));
{% endhighlight %}

{% highlight php startinline %}
// Contact: Only items that has a specific contact set
$contact_field_id = 4;

// profile_id of contact to filter on
$filter_target_profile_id = 123456789;

$collection = PodioItem::filter(123, array(
  'filters' => array(
    $contact_field_id => array($filter_target_profile_id)
  ),
));
{% endhighlight %}

{% highlight php startinline %}
// Date: Has several ways to filter.
// See https://developers.podio.com/doc/filters
$date_field_id = 4;

// Filter on absolute dates:
$collection = PodioItem::filter(123, array(
  'filters' => array(
    $date_field_id => array(
      "from" => "2014-01-01 00:00:00",
      "to" => "2014-01-31 23:59:59"
    )
  ),
));
{% endhighlight %}

## Field reference
Below you'll find examples for getting and setting field values for each of the fields available in Podio.

### App reference field

#### Getting values
Values are returned as a PodioCollection of PodioItem objects:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'app-reference';
$collection = $item->fields[$field_id]->values;

foreach ($collection as $referenced_item) {
  print "Referenced item: ".$referenced_item->title;
}
{% endhighlight %}

#### Setting values
Setting a single value can be done by setting values to a single PodioItem object or by passing in an associative array of the item structure. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'app-reference';

// Set using object
$item->fields[$field_id]->values = new PodioItem(array('item_id' => 456));

// Set using associative array
$item->fields[$field_id]->values = array('item_id' => 456);
{% endhighlight %}

When setting multiple values you can use a PodioCollection or an array of associative arrays. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'app-reference';

// Set using object
$item->fields[$field_id]->values = new PodioCollection(array(
  new PodioItem(array('item_id' => 456)),
  new PodioItem(array('item_id' => 789))
));

// Set using associative array
$item->fields[$field_id]->values = array(
  array('item_id' => 456),
  array('item_id' => 678)
);
{% endhighlight %}

------------------------------------------------------------------------------

### Calculation field
#### Getting values
Value is provided as a string with four decimals. It's often nicer to use `humanized_value()` which formats the number:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'calculation';

print $item->fields[$field_id]->values; // E.g. 123.5600
print $item->fields[$field_id]->humanized_value(); // E.g. 123.56
{% endhighlight %}

Calculation fields are read-only. It's not possible to modify the value.

------------------------------------------------------------------------------

### Category field & Question field
#### Getting values
Category and Question fields function in the same manner. Values are provided as an array of options.

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'category';

foreach ($item->fields[$field_id]->values as $option) {
  print "Option text: ".$option['text'].'. Option id: '.$option['id'];
}

{% endhighlight %}

#### Setting values
Set a single value by using the option_id. You can also add a value with `add_value()`
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'category';

// Set value to a single option
$item->fields[$field_id]->values = 4; // option_id=4

// Add value to existing selection
$item->fields[$field_id]->add_value(4); // option_id=4
{% endhighlight %}

Use an array to set multiple values
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'category';

$item->fields[$field_id]->values = array(4,5,6); // option_ids: 4, 5 and 6
{% endhighlight %}


------------------------------------------------------------------------------

### Contact field
#### Getting values

Values are returned as a PodioCollection of PodioContact objects:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'contact';
$collection = $item->fields[$field_id]->values;

foreach ($collection as $contact) {
  print "Contact: ".$contact->name;
}
{% endhighlight %}

#### Setting values
Setting a single value can be done by setting values to a single PodioContact object or by passing in an associative array of the contact structure. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'contact';

// Set using object
$item->fields[$field_id]->values = new PodioContact(array('profile_id' => 456));

// Set using associative array
$item->fields[$field_id]->values = array('profile_id' => 456);
{% endhighlight %}

When setting multiple values you can use a PodioCollection or an array of associative arrays. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'contact';

// Set using object
$item->fields[$field_id]->values = new PodioCollection(array(
  new PodioContact(array('profile_id' => 456)),
  new PodioContact(array('profile_id' => 789))
));

// Set using associative array
$item->fields[$field_id]->values = array(
  array('profile_id' => 456),
  array('profile_id' => 678)
);
{% endhighlight %}


------------------------------------------------------------------------------

### Date field
#### Getting values
Date field values have two components: The start date and the end date. You can access these through special properties, both are PHP DateTime objects. Often you'll use `humanized_value()` which provides a formatted value.
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'date';

print $item->fields[$field_id]->start; // E.g. DateTime or null
print $item->fields[$field_id]->end; // E.g. DateTiem or null
print $item->fields[$field_id]->humanized_value; E.g. "2014-02-14 14:00-15:00"
{% endhighlight %}

#### Setting values
You can simply assign values to `start` and `end` properties to modify the value. Both DateTime objects and strings are accepted.
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'date';

// Set start date using DateTime
$item->fields[$field_id]->start = new DateTime(
  DateTime::createFromFormat('Y-m-d H:i:s', '2012-12-24 14:00:00', new DateTimeZone("UTC"))
);

// Set start date using strings in various forms
$item->fields[$field_id]->start = "2012-12-24";

$item->fields[$field_id]->start = "2012-12-24 14:00:00";
{% endhighlight %}



------------------------------------------------------------------------------


### Duration field

#### Getting values
Progress fields return a simple integer representing the duration in seconds. Often you will want to use `humanized_value()` for a formatted display. You can use `hours()`, `minutes()` and `seconds()` to avoid doing your own math.
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'duration';

print $item->fields[$field_id]->values; // E.g. 3604 for one hour and 4 seconds
print $item->fields[$field_id]->humanized_value(); // E.g. "01:00:04"

print $item->fields[$field_id]->hours(); // E.g. 1
print $item->fields[$field_id]->minutes(); // E.g. 0
print $item->fields[$field_id]->seconds(); // E.g. 4

{% endhighlight %}

#### Setting values
Simply assign a new integer to set the value
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'duration';

// Set using object
$item->fields[$field_id]->values = 75; // One minute and 15 seconds ((60*1)+15)
{% endhighlight %}


------------------------------------------------------------------------------

### Image field
#### Getting values

Values are returned as a PodioCollection of PodioFile objects:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'image';
$collection = $item->fields[$field_id]->values;

foreach ($collection as $file) {
  print "File id: ".$file->file_id;
  print "File URL: ".$file->link;
}
{% endhighlight %}

You can download the files as usual

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'image';

// Download the first image
$file = $item->fields[$field_id]->values[0];
$file_content = $file->get_raw();
file_put_contents("/path/to/file".$file->name, $file_content);
{% endhighlight %}

#### Setting values
Setting a single value can be done by setting values to a single PodioFile object or by passing in an associative array of the file structure. You have to upload a file to get a file_id to use. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'image';

// Upload file
$file = PodioFile::upload("/path/to/file", "sample.jpg");

// Set using object
$item->fields[$field_id]->values = $file;

// Set using associative array
$item->fields[$field_id]->values = array('file_id' => $file->file_id);
{% endhighlight %}

When setting multiple values you can use a PodioCollection or an array of associative arrays. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'image';

// Upload files
$file_a = PodioFile::upload("/path/to/file_a", "sample_a.jpg");
$file_b = PodioFile::upload("/path/to/file_b", "sample_b.jpg");

// Set using object
$item->fields[$field_id]->values = new PodioCollection(array(
  $file_a,
  $file_b
));

// Set using associative array
$item->fields[$field_id]->values = array(
  array('file_id' => 456),
  array('file_id' => 678)
);
{% endhighlight %}


------------------------------------------------------------------------------

### Link/Embed field

#### Getting values

Values are returned as a PodioCollection of PodioEmbed objects:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'embed';
$collection = $item->fields[$field_id]->values;

foreach ($collection as $embed) {
  print "Embed id: ".$embed->embed_id;
  print "Embed URL: ".$embed->original_url;
}
{% endhighlight %}

#### Setting values
Setting a single value can be done by setting values to a single PodioEmbed object or by passing in an associative array of the embed structure. You will need to create the embed first. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'embed';

// Create embed
$embed = PodioEmbed::create(array('url' => 'http://example.com/'));

// Set using object
$item->fields[$field_id]->values = $embed;

// Set using associative array
$item->fields[$field_id]->values = array('embed_id' => $embed->embed_id);
{% endhighlight %}

When setting multiple values you can use a PodioCollection or an array of associative arrays. The following are identical:

{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'embed';

// Create embeds
$embed_a = PodioEmbed::create(array('url' => 'http://example.com/'));
$embed_b = PodioEmbed::create(array('url' => 'http://podio.com/'));

// Set using object
$item->fields[$field_id]->values = new PodioCollection(array(
  $embed_a,
  $embed_b
));

// Set using associative array
$item->fields[$field_id]->values = array(
  array('embed_id' => 456),
  array('embed_id' => 789)
);
{% endhighlight %}


------------------------------------------------------------------------------

### Location/Google Maps field

#### Getting values
Location fields return an array of strings (addresses)
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'location';

foreach ($item->fields[$field_id]->values as $location) {
  print "Location: " . $location;
}
{% endhighlight %}

#### Setting values
Set values using an array of locations
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'location';

// Set using object
$item->fields[$field_id]->values = array(
  "650 Townsend St., San Francisco, CA 94103",
  "Vesterbrogade 34, 1620 Copenhagen, Denmark"
);
{% endhighlight %}


------------------------------------------------------------------------------

### Money field

#### Getting values
Money field values have two components: The amount and the currency. You can access these through special properties. The amount is a string in order to support very large numbers. Often you'll use `humanized_value()` which provides a formatted value.
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'money';

print $item->fields[$field_id]->currency; // E.g. "USD"
print $item->fields[$field_id]->amount; // E.g. "123.5400"
print $item->fields[$field_id]->humanized_value; E.g. "$123.54"

{% endhighlight %}

#### Setting values
You can simply assign values to `currency` and `amount` properties to modify the value.
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'money';

// Set value
$item->fields[$field_id]->currency = "EUR";
$item->fields[$field_id]->amount = "456.00";
{% endhighlight %}


------------------------------------------------------------------------------

### Number field

#### Getting values
The value of a number field is a string in order to support very large numbers. Use `humanized_value()` to get a formatted string.
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'number';

print $item->fields[$field_id]->values; // E.g. 123.5600
print $item->fields[$field_id]->humanized_value(); // E.g. 123.56
{% endhighlight %}

#### Setting values
Simply assign a new string to set the value. Use a period "." as the decimal point
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'number';

// Set using object
$item->fields[$field_id]->values = "456.89";
{% endhighlight %}


------------------------------------------------------------------------------

### Progress field

#### Getting values
Progress fields return a simple integer between 0 and 100.
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'progress';

print $item->fields[$field_id]->values; // E.g. 55
{% endhighlight %}

#### Setting values
Simply assign a new integer to set the value
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'progress';

// Set using object
$item->fields[$field_id]->values = 75;
{% endhighlight %}


------------------------------------------------------------------------------

### Text field

#### Getting values
Text fields return a regular string
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'text';

print $item->fields[$field_id]->values;
{% endhighlight %}

#### Setting values
Simply assign the new string to set the value
{% highlight php startinline %}
$item = PodioItem::get_basic(123);
$field_id = 'text';

// Set using object
$item->fields[$field_id]->values = 'This is the new value for the field';
{% endhighlight %}
