Useful Tips
===========

## Hook routine

### Hook

When and where the magic happens. Position is important for when it is going to get called.

* __Action:__ `do_action( 'wp_head' );`
* __Filter:__ `apply_filters( 'the_content', $content );`

### Action or filter

Your action/filter function, what will perform the action or filter the content.

#### Action
```
function noindex() {
	// If the blog is not public, tell robots to go away.
	if ( '0' == get_option('blog_public') )
	wp_no_robots();
}
```

#### Filter

Needs at least one argument to be returned later.

```
function wpautop( $pee, $br = true ) {
	// …
	return $pee;
}
```

### To hook (calling the function)

The actual calling the function, or applying it to WP connecting it with the hook.

#### Action

`add_action( 'wp_head', 'noindex', 1

#### Action

`add_filter( 'the_content', 'wpautop' );`

### You're already using

The core is filled with hooks. Ex:

```
function wp_head() {
/**
* Print scripts or data in the head tag on the front end.
*
* @since 1.5.0
*/
do_action( 'wp_head' );
}
```

And later:

```
add_action( 'wp_head','_wp_render_title_tag', 1);
add_action( 'wp_head','wp_enqueue_scripts', 1);
add_action( 'wp_head','feed_links', 2);
add_action( 'wp_head','feed_links_extra', 3);
add_action( 'wp_head','rsd_link');
add_action( 'wp_head','wlwmanifest_link');
add_action( 'wp_head','adjacent_posts_rel_link_wp_head', 10, 0 );
add_action( 'wp_head','locale_stylesheet');
add_action( 'wp_head','noindex', 1);
add_action( 'wp_head','print_emoji_detection_script', 7);
add_action( 'wp_head','wp_print_styles', 8);
add_action( 'wp_head','wp_print_head_scripts', 9);
add_action( 'wp_head','wp_generator');
add_action( 'wp_head','rel_canonical');
add_action( 'wp_head','wp_shortlink_wp_head', 10, 0 );
add_action( 'wp_head','wp_site_icon', 99);
```

Or:

```
function the_content( $more_link_text = null, $strip_teaser = false) {
$content = get_the_content( $more_link_text, $strip_teaser );

// …
$content = apply_filters( 'the_content', $content );
$content = str_replace( ']]>', ']]&gt;', $content );
echo $content;
}
```

And:

```
add_filter( 'the_content', 'wptexturize' );
add_filter( 'the_content', 'convert_smilies' );
add_filter( 'the_content', 'convert_chars' );
add_filter( 'the_content', 'wpautop' );
add_filter( 'the_content', 'shortcode_unautop' );
add_filter( 'the_content', 'prepend_attachment' );
```

`the_content()` has filters applied, `get_the_content()` has not. The content is used in listings as well in single pages.

### Parameters

There are common parameters used in a hook routine: the name of the hook, the name of the function, the priority and the arguments passed to the function.

#### Naming the hooks

* Actions must have the moment of the call, e.g., `before_delete_post`, `delete_post`, `deleted_post` and `after_delete_post`.
* Choose a unique prefix, to avoid clashes with other core, theme or plugin functions and to permit easy identification of the functions:
	* `wpseo_`;
	* `genesis_`;
	* `advanced_ads_`.
* Dynamic names can be used to facilitate re-use of hooks as in `get_options()`:
	* `'pre_option_' . $option`
	* `'default_option_' . $option`
	* `'option_' . $option`
* All: Applies to __ALL__. Not much use except for debug.

#### PHP default functions

`add_filter( 'the_title', 'ucwords' );`

#### Priority

* Sets order of execution
* Defaults to 10
* Same priority functions are executed in the order they are registered.

#### Checking if action happened / is happening

* `did_action();`
* `doing_action();`

#### Checking if filter is applied

`has_filter( $filter[, $function] );`

* __$filter:__ name of the filter being checked. If $function is not passed, returns true if any filter is applied or false if there is no filter;
* __$function:__ name of the function to check if applied. Returns the priority in case it is applied or false in case it isn't.

#### Arguments

* Filters and actions can have arguments;
* The `int` after the priority is the number of arguments expected. Eg: `add_action( 'delete_term', '_wp_delete_tax_menu_item', 10, 3 );`
* The number of arguments is optional only if 0 or 1.
* If number of arguments is set, priority must also be set.

#### Returning

Hooks can return something, and a filter function that returns void is actually removing the filter.

### Removing Actions or Filters

* `remove_filter( 'wp_mail', 'wp_staticize_emoji_for_email' );`
* `remove_filter( 'the_content', 'wpautop' );`
* both accept:
	* hook
	* function
	* priority

## Where the hell?

Onde encontrar?:

* [Adam Brown’s hook database](http://adambrown.info/p/wp_hooks);
* [Plugin API](http://codex.wordpress.org/Plugin_API);
* Core;
* Docs.

## Debugging

* [Listing all hooks](https://www.smashingmagazine.com/2009/08/10-useful-wordpress-hook-hacks/#10-list-all-hooked-functions)
* `current_filter()` and `current_action()`
* Plugins, such as [Debug Objects](https://wordpress.org/plugins/debug-objects/)


Guide for non-dev
=================

WP is a collection of functions, database queries the work together to output HTML. Dois tipos:

* __Actions:__ Add extra functionality to a specific position in the page, ie, a message before the main content;
* __Filters:__ Intercept and modify data while it is being processed, ie, modify a page block.

In sum, actions do stuff, filters modify stuff.

## Exemples

### Actions

```
// Add some text after the header
add_action( '__after_header' , 'add_promotional_text' );
function add_promotional_text() {
	// If we're not on the home page, do nothing
	if ( !is_front_page() )
		return;
	// Echo the html
	echo "<div>Special offer! June only: Free chocolate for everyone!</div>";
}
```

Where in `header.php` there is a `do_action ( '__after_header' );`. The function will run at this precise point in the code and add this information to the page.

### Filter

```
// Change url that is linked from logo
add_filter( 'tc_logo_link_url', 'change_site_main_link' );
function change_site_main_link() {
	return 'http://example.com';
}
```

Where in `header.php` there is `apply_filters( ‘tc_logo_link_url’, esc_url( home_url( ‘/’ ) ) )`. The function get this data (the link) and returns another one (exemple.com).

## Why?

* Change almost anything in WordPress—even quite fundamental things—because a lot of WordPress’s core functions use actions and filters;
* Make changes easily: once you’ve understood the concepts, you can make some incredibly complex changes very quickly;
* Change a theme's behaviour at the source, rather than trying to retro-fit an inappropriate solution with HTML and CSS;
* Make your own changes easy to understand and easier to debug, because your code is reduced to a minimum;
* Enable and disable your changes easily because each piece of code is a small unit in your functions.php;
* Make your changes relatively upgrade-proof because you no longer need to edit or copy WordPress or any themes and plugins core files;
* Share your knowledge and swap code snippets with others.


Quick (in depth)
================

Hooks permit:

* Change default funcionality
* Add funcionality

All without changing core files. Changes in core are overwritten in updates.

## Actions

Exemple in `wp_publish_post()`:

```
function wp_publish_post( $post ) {
// Stuff that actually published the post
do_action( 'wp_insert_post', $post->ID, $post, true );
}
```

After post is published the do action calls the extra functionality hooked to it. The first parameter is the name of the hook. Example usage:

```
add_action( 'wp_insert_post', 'email_post_author', 10, 3 );
function email_post_author( $post_id, $post, $update ) {
	$email = 'mymail@mail.com';
	$subject = 'New Post Published';
	$message = 'A new post was published, use this link to view it: ' . get_permalink( $post->ID );
	wp_mail( $email, $subject, $message );
}
```

## Filters

Example in `wp_login.php`:

```
if ( ! empty( $errors ) ) {
/**
* Filter the error messages displayed above the login form.
*
* @since 2.1.0
*
* @param string $errors Login error message.
*/
echo '<div id="login_error">' . apply_filters( 'login_errors', $errors ) . "</div>\n";
}
```

Example usage:

```
add_filter( 'login_errors', 'modify_login_errors' );
function modify_login_errors() {
	return 'Login unsuccessful, try again';
}
```

### `add_action()` and `add_filter()` parameters

* __tag:__ Defines where to hook, when to execute our functions;
* __name:__ The name of the function we are calling;
* __priority:__ Priority of execution of our function, defines the order when multiple functions are attached to the same hook;
* __parameters:__ Number of parameters passed to the function. Defaults to 1, but some hooks accept more parameters.

### Tag

String the stating the point in which the hooked function will execute. There are many hooks, [Hook reference](http://codex.wordpress.org/Plugin_API/Hooks) is a fine place to start.

_Note:_ There are some variable hooks, like `{old_status}_to_{new_status}`, that means there area a number of hooks defined for transition of status, ie, `publish_to_trash` when a published post is trashed, `future_to_draft`, etc.

### Hooked function

A string with the exact name of the function to run, follows PHP rules for function names. Must be unique. Good practice dictates it must have a prefix to avoid collisions.

### Priority

The lower the number, the sooner it will execute. Sometimes it is not relevant, sometimes it is essential.

Attention must be paid specially to filters, since when modifying content the order can change things in ways not predicted.

### Parameters

The number of arguments that will be passed to the function. In most cases it must be looked upon in the source code.

Examples:

`do_action( 'wp_insert_post', $post->ID, $post, true );` (3 arguments)
`apply_filters( 'login_errors', $errors );` (1 argument)

## Creating our own

We can create our hooks using the `do_action()` and `apply_filter()` functions.


Understanding WP Hooks
======================

Location is important: when WP is running and encounters a hook, it calls and execute all functions associated with it. Ie: to call it before or after a save_post.

## Finding hooks locations

Simple test:

```
add_action( 'save_post', 'tweet_my_mood' );
function tweet_my_mood( $post_id, $post ) {
	echo get_post_meta( $post_id, 'mood', true );
	exit();
}
```

Go to codex where there will be indication of the hook location on the core files. In this case, 'Triggered by `wp_insert_post` and `wp_publish_post` in `wp-includes/post.php`.'

## Finding hook parameters


Sometimes there you know how many parameters are called, but don't know what those are. The solution here is go to codex/source again. Example: `get_post_class` has no docs, but points to the source. In source:

```
/**
* Filter the list of CSS classes for the current post.
*
* @since 2.7.0
*
* @param array $classes An array of post classes.
* @param string $class A comma-separated list of additional classes added to the post.
* @param int $post_id The post ID.
*/

$classes = apply_filters( 'post_class', $classes, $class, $post->ID );
```
