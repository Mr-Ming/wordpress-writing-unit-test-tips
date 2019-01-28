# Tips to Wordpress unit test

### How to mock function calls without dependency injection?

First Install Mockery -> `composer require --dev mockery/mockery`

Then you can use the code below

```
$clientMock = \Mockery::mock('overload:<className>');
$clientMock
	->shouldReceive('<function being mocked>')
	->andReturn('<mock result>');
```      

### How to fix `Could not load mock XYZ, class already exists`?

```
* @runInSeparateProcess
* @preserveGlobalState disabled
```
^ Looking to find better solution, this one slows down performance considerably

### Some function that cannot be unit / integration tested?
1. Private / Protected Method (but can be `hacked` using Reflection Method to overwrite method types)
2. Method that terminates a script that uses a language construct / reserved keyword

### Testing CMB2 without using function mocking?
1. Install CMB2/CMB2 for test -> `composer require --dev cmb2/cmb2`
2. Added to plugin file so it can load
```
if ( file_exists( trailingslashit( dirname( __FILE__ ) ) . 'lib/cmb2/init.php' ) && ! class_exists( 'CMB2' ) ) {
	require_once trailingslashit( dirname( __FILE__ ) ) . 'lib/cmb2/init.php';
}
```
3. Write test

Given:
```
$sample_checkbox = new_cmb2_box(
[
  'id'           => 'id123',
  'title'        => __( 'Sample Checkbox', 'cmb2' ),
  'object_types' => [ 'post', 'page' ],
  'context'      => 'normal',
  'priority'     => 'low',
]
]);
$sample_checkbox->add_field( array(
  'name' => 'Disable Comments',
  'id'   => 'disable_comments_checkbox',
  'type' => 'checkbox',
) );

```
Test:
```
$cmb2_properties = cmb2_get_metabox( 'id123' )->properties;
$this->assertEquals( 'id123', $cmb2_properties['id'] );
$this->assertEquals( 'Sample Checkbox', $cmb2_properties['title'] );
$this->assertEquals( [ 'post', 'page' ], $cmb2_properties['object_types'] );
$this->assertEquals( 'normal', $cmb2_properties['context'] );
$this->assertEquals( 'low', $cmb2_properties['priority'] );
$this->assertEquals([
  'disable_comments_checkbox' => [
    'name'          => 'Disable Comments',
    'id'            => 'disable_comments_checkbox',
    'type'          => 'checkbox',
  ],
], $cmb2_properties['fields']);
```

### Testing function that output directly into the browser html?
Given:
```
<some function foo() that does this>
echo '<div id="universal_taxonomy_container">`
echo 'something';
echo '</div>';
```

Test:
```
<some function test_foo() that will do this>
ob_start();
foo();
$actualHtml = ob_get_contents();
ob_end_clean()

$expectedHtml = '
  <div id="universal_taxonomy_container">something</div>
';

$this->assertEquals(
  $preg_replace("/\s*/", "", $expectedHtml),
  $preg_replace("/\s*/", "", $actualHtml)
);
```
