# subl:// URL handler

A subl:// _URL handler_ for **Sublime Text 2** and **Sublime Text 3** on Mac OSX. Made _very_ quickly in AppleScript. 
*If you're a Windows user, try [this Windows Sublime Text subl:// URL handler](https://github.com/ktunkiewicz/subl-handler).*

Follows the syntax of the [TextMate URL scheme](http://blog.macromates.com/2007/the-textmate-url-scheme/).

Gloms onto the following URL schemes:

- **subl://**
- **sublime://**
- **txmt://** (compatibility with existing PHP debug tools)

## Installation

- Download and unzip [Subl.app.zip](https://github.com/dhoulb/subl/releases/download/v1.2/Subl.app.zip)
- Move Subl.app to your **/Applications** directory
- Open once to register the handler

## How to use

Generate URLs in the following format:
`subl://open?url=file://{{file}}&line={{line}}&column={{column}}`

Examples:
- `subl://open?url=file:///path/to/file.php&line=12&column=4`
- `subl://open?url=file:///path/to/other.php&line=192`

### Args

- `url` - Absolute file:// link to the file, e.g. `file:///path/to/file.txt`
- `line` - Line number in the file, e.g. `192`
- `column` - Column number in the file, e.g. `14`

`url` is the only required argument. All others are optional. If you don’t specify a `url`, Subl.app will throw an error dialog.

### Path

Subl looks for your Sublime Text app in three locations in the following order:

1. `/Applications/Sublime Text 3.app`
2. `/Applications/Sublime Text 2.app`
3. `/Applications/Sublime Text.app`

_As a quick fix, if your Sublime Text app isn't in any of these locations, you can just rename and move it so it does match one of them._

## Sublime Text and PHP errors

This code snippet will re-route all PHP errors to a custom error handler, and turn filenames into clicky Sublime Text links.

```php
// Custom error handler function.
// Turns filenames into clicky subl:// links (requires http://github.com/dhoulb/subl)
// Also cleanses the ugly part of pathnames, so "/long/path/to/file/name.php on line 7" becomes "name.php - 7"
function subl_handler($level, $error, $file, $line)
{
	// Check if error reporting is off.
	if ($level === 0) return;

	// Is there a file?
	if ($file)
	{
		// Ugly part to strip off the path name.
		$strip = dirname($_SERVER['SCRIPT_FILENAME']) . DIRECTORY_SEPARATOR;

		// Change $file to a clicky link.
		$file = '<strong><a href="subl://open?url=file://' . $file . '&line=' . $line . '">'
		. str_replace($strip, '', $file)
		. ' - ' . $line
		. '</a></strong>';
	}

	// Output the file and the error.
	echo '<pre class="error">';
	if ($file) echo $file . '<br>';
	echo $error;
	echo '</pre>';
}

// Additional function to enable handling for fatal errors too.
function subl_shutdown() { if ($error = error_get_last()) subl_handler($error['type'], $error['message'], $error['file'], $error['line']); }

// Register handlers.
set_error_handler('subl_handler', E_ALL); // Register custom function as the error handler.
register_shutdown_function('subl_shutdown'); // Register the custom shutdown function.
ini_set('display_errors', false); // Turn off built-in errors.
```

_Remember:_ It's recommended to show no errors at all on live production websites. PHP errors leak details than can be useful to attackers.

## Sublime Text and Ruby on Rails errors

Install Subl.app as above, and include [Better Errors](http://sublimetext.userecho.com/topic/97042-url-sheme-support-subletc/#comment_263670) in your Rails application. 

Use the following code snippet to tell Better Errors to use Subl.app as its editor. Change `local_path` to the full base path of your application.

```ruby
if defined? BetterErrors
  BetterErrors.editor = proc { |full_path, line|
    full_path = full_path.sub(Rails.root.to_s, local_path)
    "subl://open?url=file://#{full_path}&line=#{line}"
  }
end
```

See the [Better Errors Wiki](https://github.com/charliesome/better_errors/wiki) for more information about pointing editor links at Sublime Text.

## General comments

Hopefully some day a URL scheme will be baked in to Sublime Text, and all of this will be unneccessary!

See [this thread](http://sublimetext.userecho.com/topics/3451-uri-scheme-handler/) on the Sublime Text support system, and upvote if you think this is important.
