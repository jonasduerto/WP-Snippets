# 1. Colored Admin Table BGs by Post Status

The administration panel is designed in a very formulaic manner, but looking from a distance, everything tends to blur together. This snippet allows for custom-colored table row backgrounds for each post depending on the status (draft, pending, private, etc).

```php
add_action('admin_footer','posts_status_color');
  function posts_status_color(){
?>
<style>
  .status-draft{background: #FCE3F2 !important;}
  .status-pending{background: #87C5D6 !important;}
  .status-publish{/* no background keep wp alternating colors */}
  .status-future{background: #C6EBF5 !important;}
  .status-private{background:#F2D46F;}
</style>
<?php
}
```
___
# 2. Pull & Cache a Recent Tweet

There are dozens of articles out there teaching how to pull Twitter followers and recent tweets with API calls. WordPress can be a little tricky and it’s different if you’re running a custom caching plugin. This code uses the Transient API to cache the query with a customized expiration time.

```php
/**
* Get the latest Tweet Text by Username
*
* @uses Transient - API for chaching
* @copyright Mathias Schopmans - 10/2010
* 
* @param string $username: Twitter Username
*/

function get_last_tweet($username='nasenmann'){	
	if (false === ($last_tweet = get_transient('last_tweet_by_'.$username))) {

		$res = wp_remote_get('http://twitter.com/statuses/user_timeline/'.$username.'.json');
		$tweets = json_decode($res['body']);

		foreach ($tweets as $tweet){
			if(empty($tweet->in_reply_to_user_id)){
				$last_tweet = $tweet->text;
				break;
			}
		}

		set_transient('last_tweet_by_'.$username, $last_tweet, 10*60); 

	}
	return $last_tweet;
}

print_r(get_last_tweet());
```
___
# 3. Automatically Clear W3 Total Cache

If you’re running a dynamic website then some of your pages may include user-submitted content. Since WordPress doesn’t include caching by default we can instead use plugins like W3 Total Cache. And without manually clearing the cache it will only expire after a set amount of time.

Manually resetting can work if you’re running a typical blog or static content website. But if you need to re-cache for every new comment, user submission, or another similar event, try out this code snippet targeting a full cache reset or an individual post/page ID reset.

```php
// purge entire cache
if(function_exists('w3tc_pgcache_flush')) {  w3tc_pgcache_flush();  }

// flush individual post/page
if (function_exists('w3tc_pgcache_flush_post')) {  w3tc_pgcache_flush_post($post_id); }
```
___
# 4. Limit or Disable Post Revisions

Your database can bloat up quickly with thousands of post revisions. Why would you need 50 different versions of the same post? Backing up your website can be dreadful with a tremendous database. Try using these snippets to either disable post revisions or limit them at a maximum number.

Please note this code should be added to your wp-config.php file in the root directory.

```php
// disable post revisions
define('WP_POST_REVISIONS', false);

// limit five post revisions
define('WP_POST_REVISIONS', 5);
5. Disable WordPress Automatic Updates

WordPress’ developers wanted to make updates easier for the typical users. Automatic updates run in the backend for small-scale releases. But if you’d rather manually perform updates on your own time, this snippet provides a quick fix.

Please note this code should be added to your wp-config.php file in the root directory.

define('WP_AUTO_UPDATE_CORE', false);
```
___
# 6. HTTPS with Non-Secure Media

Loading images or media through an external server like MaxCDN is often quicker, but doesn’t always include HTTPS support. If your website is serving SSL secured pages then you won’t get the green padlock or blue address bars. Instead you get the message that some items on the page aren’t delivered securely.

Granted there are paid solutions for SSL media delivery, but this would cost extra money which not everyone has available. Instead you might try this code snippet which forces any SSL pages with non-secure media to be delivered as HTTP.

```php
add_action('wp_head','nocdn_on_ssl_page');
function nocdn_on_ssl_page() {
    if($_SERVER['HTTPS'] == "on") {
        define('DONOTCDN', true);
    }
}
```
___
# 7. Require a Featured Post Image

If you’re running a larger blog/magazine which needs a featured image on each post, this snippet is an amazing solution. New writers may try to publish articles without any thumbnail image because they either forgot or don’t care.

This code checks for a featured image when updating. If no image exists it will save the current draft without publishing, and display an error message to the writer. This will ensure that no post can go live without a featured image attached.

```php
add_action('save_post', 'wpds_check_thumbnail');
add_action('admin_notices', 'wpds_thumbnail_error');
function wpds_check_thumbnail($post_id) {
    // change to any custom post type
    if(get_post_type($post_id) != 'post')
        return;
    if ( !has_post_thumbnail( $post_id ) ) {
        // set a transient to show the users an admin message
        set_transient( "has_post_thumbnail", "no" );
        // unhook this function so it doesn't loop infinitely
        remove_action('save_post', 'wpds_check_thumbnail');
        // update the post set it to draft
        wp_update_post(array('ID' => $post_id, 'post_status' => 'draft'));
        add_action('save_post', 'wpds_check_thumbnail');
    } else {
        delete_transient( "has_post_thumbnail" );
    }
}
function wpds_thumbnail_error()
{
    // check if the transient is set, and display the error message
    if ( get_transient( "has_post_thumbnail" ) == "no" ) {
        echo "<div id='message' class='error'><p><strong>You must select Featured Image. Your Post is saved but it can not be published.</strong></p></div>";
        delete_transient( "has_post_thumbnail" );
    }
}
```
___
# 8. Add Custom Image Sizes

Uploaded images can be embedded into a post using custom sizes. WordPress comes out-of-the-box with a number of default image sizes. But you can also create your own dimensions which may be selected from the post editor.

```php
if ( function_exists( 'add_image_size' ) ) {
    add_image_size( 'new-size', 300, 100, true ); //(cropped)
}
add_filter('image_size_names_choose', 'my_image_sizes');
function my_image_sizes($sizes) {
        $addsizes = array(
                "new-size" => __( "New Size")
                );
        $newsizes = array_merge($sizes, $addsizes);
        return $newsizes;
}
```
___
# 9. Automatic Media Shortcodes

The WordPress WYSIWYG editor comes with a lot of great functionality. Adding this snippet allows writers to select different shortcodes from a dropdown list. You can quickly embed media or write custom functions to play out in areas of post content.

```php
add_action('media_buttons','add_sc_select',11);
function add_sc_select(){
    global $shortcode_tags;
     /* ------------------------------------- */
     /* enter names of shortcode to exclude bellow */
     /* ------------------------------------- */
    $exclude = array("wp_caption", "embed");
    echo '&nbsp;<select id="sc_select"><option>Shortcode</option>';
    foreach ($shortcode_tags as $key => $val){
            if(!in_array($key,$exclude)){
            $shortcodes_list .= '<option value="['.$key.'][/'.$key.']">'.$key.'</option>';
            }
        }
     echo $shortcodes_list;
     echo '</select>';
}
add_action('admin_head', 'button_js');
function button_js() {
        echo '<script type="text/javascript">
        jQuery(document).ready(function(){
           jQuery("#sc_select").change(function() {
                          send_to_editor(jQuery("#sc_select :selected").val());
                          return false;
                });
        });
        </script>';
}
```
___
# 10. Remove WordPress Version from Header

For some webmasters, the WordPress version number is private information. It can be a security concern to declare which version of WordPress you’re running. This brief snippet removes all version info from the head section of each page.

```php
// remove version info from head and feeds
function complete_version_removal() {
    return '';
}
add_filter('the_generator', 'complete_version_removal');
```
___
# 11. Update WordPress Search Permalink URL

Even the pretty permalinks in WordPress don’t update the GET-style search results. Thankfully this handy snippet can solve the problem by forcing a /search/keywords+here type of URL structure.

```php
function fb_change_search_url_rewrite() {
	if ( is_search() && ! empty( $_GET['s'] ) ) {
		wp_redirect( home_url( "/search/" ) . urlencode( get_query_var( 's' ) ) );
		exit();
	}	
}
add_action( 'template_redirect', 'fb_change_search_url_rewrite' );
```
___
# 12. Sharpen Uploaded Images when Resized

No matter how clear your photo is when uploaded, WordPress always seems to automate somewhat blurry thumbnails. This PHP snippet will update the resizing function to force more sharpness out of each resized thumbnail.

```php
function ajx_sharpen_resized_files( $resized_file ) {

    $image = wp_load_image( $resized_file );
    if ( !is_resource( $image ) )
        return new WP_Error( 'error_loading_image', $image, $file );

    $size = @getimagesize( $resized_file );
    if ( !$size )
        return new WP_Error('invalid_image', __('Could not read image size'), $file);
    list($orig_w, $orig_h, $orig_type) = $size;

    switch ( $orig_type ) {
        case IMAGETYPE_JPEG:
            $matrix = array(
                array(-1, -1, -1),
                array(-1, 16, -1),
                array(-1, -1, -1),
            );

            $divisor = array_sum(array_map('array_sum', $matrix));
            $offset = 0; 
            imageconvolution($image, $matrix, $divisor, $offset);
            imagejpeg($image, $resized_file,apply_filters( 'jpeg_quality', 90, 'edit_image' ));
            break;
        case IMAGETYPE_PNG:
            return $resized_file;
        case IMAGETYPE_GIF:
            return $resized_file;
    }

    return $resized_file;
}   

add_filter('image_make_intermediate_size', 'ajx_sharpen_resized_files',900);
```
___
# 13. Remove Default Image Thumbnails

When uploading a new image, WordPress automatically crops a number of extra thumbnails. By disabling this feature you save on excess server processing along with space on the hard drive. Some people prefer to have separate thumbnail images but it’s not always necessary – especially when you’re looking to save space in regards to website backups.

```php
update_option( 'thumbnail_size_h', 0 );
update_option( 'thumbnail_size_w', 0 );
update_option( 'medium_size_h', 0 );
update_option( 'medium_size_w', 0 );
update_option( 'large_size_h', 0 );
update_option( 'large_size_w', 0 );
```
___
# 14. Custom Author Slug URL Base

You should be familiar with the WordPress author archive located at /author/username/. The other permalink structures are easily editable, but for some reason the author base requires some customization. I hope this code snippet will provide the answer to curious developers.

```php
add_action('init', 'cng_author_base');
function cng_author_base() {
    global $wp_rewrite;
    $author_slug = 'profile'; // change slug name
    $wp_rewrite->author_base = $author_slug;
}
```
___
# 15. Include Post/Page IDs in Admin Table

How many times have you needed to search for an individual post or page ID? This snippet will include the ID number directly into the admin table list for posts and pages. It does take up a bit of extra space, but you can always remove other default columns if needed.

```php
add_filter('manage_posts_columns', 'posts_columns_id', 5);
add_action('manage_posts_custom_column', 'posts_custom_id_columns', 5, 2);
add_filter('manage_pages_columns', 'posts_columns_id', 5);
add_action('manage_pages_custom_column', 'posts_custom_id_columns', 5, 2);

function posts_columns_id($defaults){
    $defaults['wps_post_id'] = __('ID');
    return $defaults;
}

function posts_custom_id_columns($column_name, $id){
    if($column_name === 'wps_post_id'){
        echo $id;
    }
}
```
___
# 16. Force SSL/HTTPS on Specific Post IDs

For checkout pages or unique signup/login pages you may wish to set HTTPS as the request header. This can be done using .htaccess but it’s also possible using a PHP code snippet. Be sure you’ve already created the page(s) and have an SSL license for the domain.

```php
function wps_force_ssl( $force_ssl, $post_id = 0, $url = '' ) {
    if ( $post_id == 25 ) {
        return true
    }
    return $force_ssl;
}
add_filter('force_ssl' , 'wps_force_ssl', 10, 3);
```
___
# 17. Automatic Twitter @mention links

If you often include Twitter accounts in post content then you might fall in love with this handy snippet. Whenever a post contains any username in the format of @username, the HTML will be customized to link directly to that Twitter profile. Pretty snazzy huh?

```php
function content_twitter_mention($content) {
	return preg_replace('/([^a-zA-Z0-9-_&])@([0-9a-zA-Z_]+)/', "$1<a href=\"http://twitter.com/$2\" target=\"_blank\" rel=\"nofollow\">@$2</a>", $content);
}

add_filter('the_content', 'content_twitter_mention');   
add_filter('comment_text', 'content_twitter_mention');
```
___
# 18. Clear the Media Upload Buttons

In some cases you may have authors or editors who know how to(and prefer to) manually include images. This block of code will clean out the media upload popup to only display the essential elements. Play around with this code snippet to see how it affects the media upload interface.

```php
function myAttachmentFields($form_fields, $post) {
    // Can now see $post becaue the filter accepts two args, as defined in the add_fitler
    if ( substr( $post->post_mime_type, 0, 5 ) == 'image' ) {
        $form_fields['image_alt']['value'] = '';
        $form_fields['image_alt']['input'] = 'hidden';
        $form_fields['post_excerpt']['value'] = '';
        $form_fields['post_excerpt']['input'] = 'hidden';
        $form_fields['post_content']['value'] = '';
        $form_fields['post_content']['input'] = 'hidden';
        $form_fields['url']['value'] = '';
        $form_fields['url']['input'] = 'hidden';
        $form_fields['align']['value'] = 'aligncenter';
        $form_fields['align']['input'] = 'hidden';
        $form_fields['image-size']['value'] = 'thumbnail';
        $form_fields['image-size']['input'] = 'hidden';
        $form_fields['image-caption']['value'] = 'caption';
        $form_fields['image-caption']['input'] = 'hidden';
        $form_fields['buttons'] = array(
            'label' => '',
            'value' => '',
            'input' => 'html'
        );
        $filename = basename( $post->guid );
        $attachment_id = $post->ID;
        if ( current_user_can( 'delete_post', $attachment_id ) ) {
            if ( !EMPTY_TRASH_DAYS ) {
                $form_fields['buttons']['html'] = "<a href='" . wp_nonce_url( "post.php?action=delete&smp;amp;post=$attachment_id", 'delete-attachment_' . $attachment_id ) . "' id='del[$attachment_id]' class='delete'>" . __( 'Delete Permanently' ) . '</a>';
            } elseif ( !MEDIA_TRASH ) {
                $form_fields['buttons']['html'] = "<a href='#' class='del-link' onclick="document.getElementById('del_attachment_$attachment_id').style.display='block';return false;">" . __( 'Delete' ) . "</a>
                         <div id='del_attachment_$attachment_id' class='del-attachment' style='display:none;'>" . sprintf( __( 'You are about to delete <strong>%s</strong>.' ), $filename ) . "
                         <a href='" . wp_nonce_url( "post.php?action=delete&amp;post=$attachment_id", 'delete-attachment_' . $attachment_id ) . "' id='del[$attachment_id]' class='button'>" . __( 'Continue' ) . "</a>
                         <a href='#' class='button' onclick="this.parentNode.style.display='none';return false;">" . __( 'Cancel' ) . "</a>
                         </div>";
            } else {
                $form_fields['buttons']['html'] = "<a href='" . wp_nonce_url( "post.php?action=trash&amp;post=$attachment_id", 'trash-attachment_' . $attachment_id ) . "' id='del[$attachment_id]' class='delete'>" . __( 'Move to Trash' ) . "</a><a href='" . wp_nonce_url( "post.php?action=untrash&amp;post=$attachment_id", 'untrash-attachment_' . $attachment_id ) . "' id='undo[$attachment_id]' class='undo hidden'>" . __( 'Undo' ) . "</a>";
            }
        }
        else {
            $form_fields['buttons']['html'] = '';
        }
    }
    return $form_fields;
}
// Hook on after priority 10, because WordPress adds a couple of filters to the same hook - added accepted args(2)
add_filter('attachment_fields_to_edit', 'myAttachmentFields', 11, 2 );
```
___
# 19. Caching Queries with the Transient API

Here’s a fantastic code snippet for caching any WordPress query using the Transient API. It’s a newer feature that allows developers to pull out unique queries from the database and cache them independently of the posts and pages. You can find plenty of great tutorials explaining how to use this API and why you’d want to.

```php
// Get any existing copy of our transient data
if ( false === ( $special_query_results = get_transient( 'special_query_results' ) ) ) {
    // It wasn't there, so regenerate the data and save the transient
     $special_query_results = new WP_Query( 'cat=5&order=random&tag=tech&post_meta_key=thumbnail' );
     set_transient( 'special_query_results', $special_query_results );
}

// Use the data like you would have normally...
```
___
# 20. Force All Scripts into wp_footer()

When considering performance speed, you may want to keep JS scripts organized beneath your overall page HTML. This allows most of the DOM to finish loading before running any dynamic scripts. It definitely won’t be useful for everyone, but this snippet can speed up WordPress sites for non-technical users.

```php
if(!is_admin()){
	remove_action('wp_head', 'wp_print_scripts');
	remove_action('wp_head', 'wp_print_head_scripts', 9);
	remove_action('wp_head', 'wp_enqueue_scripts', 1);

	add_action('wp_footer', 'wp_print_scripts', 5);
	add_action('wp_footer', 'wp_enqueue_scripts', 5);
	add_action('wp_footer', 'wp_print_head_scripts', 5);
}
```
___
# 21. Custom WordPress User Roles

Have you ever wanted a role which is higher than editor but not an administrator? Or maybe an author with extra benefits, but not quite an editor of the whole website?

This code snippet should only be run once and then commented out or deleted. It’s a code snippet which injects a new user role into your WordPress database, and defines all the unique privileges as well.

```php
// To add the new role, using 'international' as the short name and
// 'International Blogger' as the displayed name in the User list and edit page:
/*
add_role('international', 'International Blogger', array(
    'read' => true, // True allows that capability, False specifically removes it.
    'edit_posts' => true,
    'delete_posts' => true,
    'edit_published_posts' => true,
    'publish_posts' => true,
    'edit_files' => true,
    'import' => true,
    'upload_files' => true //last in array needs no comma!
));
*/
// To remove one outright or remove one of the defaults:
/* 
remove_role('international');
*/
```
___
# 22. Block Dashboard except for Admin Users

If your WordPress site encourages new user signups then this may prevent headaches dealing with backend access. This code snippet helps when you want users to signup but don’t wish to give them access to the dashboard. Anyone beneath an administrator will be automatically redirected to the homepage.

```php
add_action( 'init', 'blockusers_init' );
function blockusers_init() {
    if ( is_admin() && ! current_user_can( 'administrator' ) ) {
        wp_redirect( home_url() );
        exit;
    }
}
```
___
# 23. Login with Username or E-mail

If you’d like some enhanced WP functionality then look no further! Most social networks allow users to login with either a username or e-mail address. WordPress doesn’t have this feature by default, although it is a quick customization without too much hassle.

```php

function login_with_email_address($username) {
    $user = get_user_by('email',$username);
    if(!empty($user->user_login))
        $username = $user->user_login;
    return $username;
}
add_action('wp_authenticate','login_with_email_address');

function change_username_wps_text($text){
    if(in_array($GLOBALS['pagenow'], array('wp-login.php'))){
        if ($text == 'Username'){$text = 'Username / Email';}
    }

    return $text;
}
add_filter( 'gettext', 'change_username_wps_text' );
```
___
# 24. Disable Links from User Comments

By default, any comment posted to your website will automatically wrap anchor links around text starting with http://. It creates automatic hyperlinks and this doesn’t often become a problem unless you constantly deal with spammers.

This snippet will force all links in comments to stay as plain text on the page. It seems like this would be a default feature in WordPress, but thankfully it’s just one line of PHP code.

```php
remove_filter('comment_text', 'make_clickable', 9);
```
___
# 25. Remove Unwanted Dashboard Items

If you’re like me then you prefer a clean interface without too many distractions. The WordPress dashboard includes a lot of extra stuff like news, latest plugins, quick draft, and other similar features. This PHP code allows you to disable any unwanted items on the dashboard panel.

```php
add_action('wp_dashboard_setup', 'my_custom_dashboard_widgets');

function my_custom_dashboard_widgets() {
global $wp_meta_boxes;
 //Right Now - Comments, Posts, Pages at a glance
unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_right_now']);
//Recent Comments
unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_recent_comments']);
//Incoming Links
unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_incoming_links']);
//Plugins - Popular, New and Recently updated WordPress Plugins
unset($wp_meta_boxes['dashboard']['normal']['core']['dashboard_plugins']);

//Wordpress Development Blog Feed
unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_primary']);
//Other WordPress News Feed
unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_secondary']);
//Quick Press Form
unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_quick_press']);
//Recent Drafts List
unset($wp_meta_boxes['dashboard']['side']['core']['dashboard_recent_drafts']);
}
```
