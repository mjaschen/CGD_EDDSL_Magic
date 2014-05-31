CGD_EDDSL_Magic
===============

A drop-in class that magically manages your EDD SL plugin licensing.

#  What is magic and why do I need it?
EDD's brilliant Software Licensing add-on is awesome, but its implementation examples are thin.  Managing the various activation and licensing states takes a good amount of research and setup. 

Once you have it setup, it can be a major pain in the tucus to manage across your various plugins. 

For example, I have 8 plugins.  Everytime I find a bug in my licensing code, I have to update 8 plugins that have 8 slightly different implementations.  It's a major headache. 

`CGD_EDDSL_Magic` fixes all of this.  With as little as a single line of code, you can add a fully functioning licensing settings page to your plugin.  

# How do I magic?

The fastest way to understand implementation of `CGD_EDDSL_Magic` is to look at our example plugin in `examples/awesome-plugin`.  It shows a basic installation.

## Here are the typical steps:

1) Copy or clone CGD_EDDSL_Magic into your plugin project.  Put it in a lib or inc folder.

2)  At the top of your main plugin file, or wherever you do your includes, add some code like:


```php
if( !class_exists( 'CGD_EDDSL_Magic' ) ) {
	// load our custom updater
	include( dirname( __FILE__ ) . '/lib/CGD_EDDSL_Magic/CGD_EDDSL_Magic.php' );
}
```

3) In your plugin constructor (or in the main plugin file if you're not using classes for some reason),  instantiate `CGD_EDDSL_Magic`. 

```php
$updater = new CGD_EDDSL_Magic($prefix, $menu_slug,  $host_url, $plugin_version, $plugin_name, $plugin_author);
```
	
## The parameters:
#### $prefix
This is a unique prefix for your instance.  It's used for saving settings and hooking up various behaviors.  Keep it short, and no spaces or weird symbols or other funny business.  Example: myplugin

#### $menu_slug
Assuming your plugin has a menu page, you would set the slug of that menu here so that `CGD_EDDSL_Magic` can add a submenu called "License" to this menu.  If you'd rather control this yourself, set to `false`. 

#### $host_url
The URL of the site that hosts your plugins. 

#### $plugin_version
The version of the plugin.

#### $plugin_name
The name of the plugin as setup in EDD.

#### $plugin_author
The author of the plugin.

If you're using a class, it's probably a good idea to set a class variable called `updater` and then assign the new `CGD_EDDSL_Magic` instance to that. It will make it easier to access later.

**In a basic setup, you're done at this point.  Your plugin will now have a fully functioning license settings page, added to whatever your parent menu is.  For more advanced options, continue below.**


# Advanced Implementation

## Controlling where the licensing page
If you would like to control where the license settings page is, that's actually really easy to do too. 

Just set `$menu_slug` to false in your instantation, and then drop this line in your admin page, wherever you prefer:
``` php
$updater->admin_page();
```
Obviously, the exact syntax will vary depending on how you implement it.  This is one reason I find it easier to set the updater instance as a class variable. 

**One important note: Do not place this line in another HTML form.  It will screw things up. Browsers hate nested forms. (and HTML standards do not permit them)**

## Cronning license checks
If you want it,  `CGD_EDDSL_Magic` includes a way to force regular license checks.  To do this, you'd add the following code to your activation or deactivation hooks:

### Activation hook
``` php
$this->updater->set_license_check_cron();
```

### Deactivation hook
``` php
$this->updater->unset_license_check_cron();
```

This will create daily checks that keep your key_status variable up-to-date. 

# Really Advanced Implementation

If this does not satisfy you, and you want to add some type of nag to the plugin listing on the plugins page in WP admin, here's a quick example of how you might do that.  This is just a starting point, so you'll have to parse through it to figure out how it works. 

```php
	add_action('admin_menu', 'add_key_nag', 11);
	function add_key_nag() {
		global $pagenow;

	    if( $pagenow == 'plugins.php' ) {
	        add_action( 'after_plugin_row_' . plugin_basename(__FILE__), 'after_plugin_row_message', 10, 2 );
	    }
	}

	function after_plugin_row_message() {
		$key_status = $this->updater->get_field_value('key_status');

		if ( empty($key_status) ) return;

		if ( $key_status != "valid" ) {
			$current = get_site_transient( 'update_plugins' );
			if ( isset( $current->response[ plugin_basename(__FILE__) ] ) ) return;

			if ( is_network_admin() || ! is_multisite() ) {
				$wp_list_table = _get_list_table('WP_Plugins_List_Table');
				echo '<tr class="plugin-update-tr"><td colspan="' . $wp_list_table->get_column_count() . '" class="plugin-update colspanchange"><div class="update-message">';
				echo keynag();
				echo '</div></td></tr>';
			}
		}
	}
	
	function keynag() {
		return "<span style='color:red'>You're missing out on important updates because your license key is missing, invalid, or expired.</span>";
	}
```

# Changelog

## Version 0.2
- Added url parameter to API requests for more reliable handling. 

## Version 0.1
- Initial release. 
