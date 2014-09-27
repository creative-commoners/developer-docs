summary: An overview of the steps involved in delivering a SilverStripe web page.

# Execution Pipeline

## Introduction

This page documents all the steps from an URL request to the delivered page.

## .htaccess and RewriteRule

Silverstripe uses **[mod_rewrite](http://httpd.apache.org/docs/2.0/mod/mod_rewrite.html)** to deal with page requests.
So instead of having your normal everyday `index.php` file which tells all, you need to look elsewhere.

The basic .htaccess file after installing SilverStripe looks like this:

	<file>
	### SILVERSTRIPE START ###
	
	<Files *.ss>
	Order deny,allow
	Deny from all
	Allow from 127.0.0.1
	</Files>
	
	<IfModule mod_rewrite.c>
	RewriteEngine On
	
	RewriteRule ^vendor(/|$) - [F,L,NC]
	RewriteRule silverstripe-cache(/|$) - [F,L,NC]
	RewriteRule composer\.(json|lock) - [F,L,NC]
	
	RewriteCond %{REQUEST_URI} ^(.*)$
	RewriteCond %{REQUEST_FILENAME} !-f
	RewriteRule .* framework/main.php?url=%1 [QSA]
	</IfModule>
	### SILVERSTRIPE END ###
	</file>

The `<Files>` section denies direct access to the template files from anywhere but the server itself.

The `<IfModule>` section enables the rewriting engine. Requests will be rewritten if they meet the following
criteria:

*  URI doesn't end in .gif, .jpg, .png, .css, or .js
*  The requested file doesn't exist on the filesystem
 
The rewriting engine then calls `framework/main.php` with the REQUEST_FILENAME (%1) as the `url` parameter and also appends the original
QUERY_STRING. **Example:** *"myblog/cakes?page=2"* is rewritten as *"framework/main.php?url=myblog/cakes&page=2"*.

See the [mod_rewrite documentation](http://httpd.apache.org/docs/2.0/mod/mod_rewrite.html) for more information on how
mod_rewrite works.


## main.php

All requests go through `main.php`, which sets up the environment and then hands control over to `Director`.

## Director and URL patterns

main.php relies on `[api:Director]` to work out which controller should handle this request.  `[api:Director]` will instantiate that
controller object and then call `[api:Controller::run()]`.

In general, the URL is build up as follows: `page/action/ID/otherID` - e.g. http://localhost/mypage/addToCart/12.
This will add an object with ID 12 to the cart.

When you create a function, you can access the ID like this:

	:::php
	public function addToCart ($request) {
		$param = $request->allParams();
		echo "my ID = " . $param["ID"];
		$obj = MyProduct::get()->byID($param["ID"]);
		$obj->addNow();
	}

## Controllers and actions

`[api:Controller]`s are the building blocks of your application.

**See:** The API documentation for `[api:Controller]`

You can access the following controller-method with /team/signup

	:::php
	class Team extends DataObject {}

	class Team_Controller extends Controller {

		private static $allowed_actions = array('signup');

		public function signup($id, $otherId) {
			return $this->renderWith('MyTemplate');
		}

	}

## SSViewer template rendering

See [templates](/reference/templates) for information on the SSViewer template system.

## Flush requests

If `?flush=1` is requested in the URL, e.g. http://mysite.com?flush=1, this will trigger a call to `flush()` on
any classes that implement the `Flushable` interface.

See [reference documentation on Flushable](/reference/flushable) for more details.

