# Slack Theming

_as the old saying goes, if you want something doing properly, you have to do it yourself_

As Slack does not currently provide a dark theme for photonophobes, there are more than a few instructions online and here in github. In my experience, they tend to include lots of very specific CSS changes which stop working regularly as Slack updates its styling.

My goal is to make the process easier, focusing primarily on the main messages portion of the screen (as Slack does let you customise the sidebar to some extent) and simple but effective changes. In theory, as the changes are much fewer in number, they should be easier to maintain and last a bit longer before failing.

My knowledge of how to do this is sourced partially from [widget-/slack-black-theme](https://github.com/widget-/slack-black-theme).

## Installation

**Please note that you are modifying a core file and will have to repeat this process each time Slack updates**

#### Step 1 Locate your slack installation folder

* Windows: `%homepath%\AppData\Local\slack\`
* Mac: `/Applications/Slack.app/Contents/`
* Linux: `/usr/lib/slack/` (Debian-based, please note that this will not work if you have installed Slack via snap for complicated application reasons and if you break slack, you can reinstall with something along the lines of `sudo apt-get install --reinstall slack-desktop`)

#### Step 2 Pick the appropriate file to edit

As of Slack 3.0.0, you need to modify the `resources\app.asar.unpacked\src\static\ssb-interop.js` file.

As of Slack 4.0.0, you need to modify the `dist\ssb-interop.bundle.js` file which is part of the `resources\app.asar` archive. The easiest way of editing is with the [Asar7z plugin](http://www.tc4shell.com/en/7zip/asar/) for [7-Zip](https://www.7-zip.org/) but you can also use npm to install asar to manage the archive.

#### Step 3 Include the code

You have to include the following code at the bottom of the file.

```js
// only do this once the whole app has loaded
document.addEventListener("DOMContentLoaded", function() {

	// since we are only interested in the textual content, we can host stylesheets at github
	var stylesheets = [
		// EITHER

		'https://raw.githubusercontent.com/willpower232/slack-theming/master/stylesheets/3/fontnunito.css',
		'https://raw.githubusercontent.com/willpower232/slack-theming/master/stylesheets/3/darkbyinversefilter.css',
		'https://raw.githubusercontent.com/willpower232/slack-theming/master/stylesheets/3/sidebarlinksrounded.css',

		// OR

		'https://raw.githubusercontent.com/willpower232/slack-theming/master/stylesheets/4/darkbyinversefilter.css',
		'https://raw.githubusercontent.com/willpower232/slack-theming/master/stylesheets/4/hidequickjump.css',
		'https://raw.githubusercontent.com/willpower232/slack-theming/master/stylesheets/4/sidebarlinksrounded.css',
	];

	// create a separate fetch for each stylesheet for asynchronicity
	var promises = [];
	stylesheets.forEach(function(stylesheet) {
		promises.push(fetch(stylesheet).then(function(response) {
			// extract the text from the response
			return response.text();
		}));
	});

	// create a central promise to combine everything
	var csspromise = Promise.all(promises).then(function(values) {
		// values is an array of the response texts
		return values.join(' ');
	}).then(function(combinedcss) {
		// now is your opportunity to add further css manually
		var customcss = `

		`;

		return combinedcss + customcss;
	});

	// insert a style tag into the wrapper view
	csspromise.then(function(allthecss) {
		var s = document.createElement('style');
		s.type = 'text/css';
		s.innerHTML = allthecss;
		document.head.appendChild(s);
	});
});
```

Please note this is set up that you can include multiple stylesheets from various sources and also add custom CSS code yourself. The stylesheets in this repository have very specific tasks so you can only include those that you want if you wish. The stylesheets are now in subfolders to appropriately cover the changes in each version.

Unfortunately for customisations, Slack 4 includes Content Security Policy so it is now more difficult to include custom code like fonts so I haven't updated the font stylesheet.

If you examine other sources, you will find they include the following section of code. I have not found it necessary to include it but YMMV so I've kept a copy here.

```js
	// insert a style tag into each webview too
	document.querySelectorAll(".TeamView webview").forEach(function(webview) {
		webview.addEventListener('ipc-message', function(message) {
			if (message.channel == 'didFinishLoading') {
				csspromise.then(function(allthecss) {
					webview.executeJavaScript(`
						var s = document.createElement('style');
						s.type = 'text/css';
						s.innerHTML = \`${allthecss}\`;
						document.head.appendChild(s);
					`);
				});
			}
		});
	});
```

## Development

As Slack is a web app, you can test out various CSS tweaks in your browser. If you add a `<style></style>` tag to the page containing all the CSS you want to use, it should have the same affect in the browser that it does in the app.

If you would like to use the Developer Tools inside of the Slack app, you'll need to start it with an environment variable as follows:

* Windows: duplicate (or edit) the start menu shortcut and change the target to `C:\Windows\System32\cmd.exe /c " SET SLACK_DEVELOPER_MENU=TRUE && start C:\existing\path\to\slack.exe"` using the path to `slack.exe` that you already have

* Mac: `export SLACK_DEVELOPER_MENU=true; open -a /Applications/Slack.app`

* Linux: `export SLACK_DEVELOPER_MENU=true && /usr/bin/slack`

The Developer Tools are also handy for confirming that your javascript modifications for installation have not caused any errors.

### Caching

The app will quite aggressively cache the stylesheets you load which is great for long term performance but not so great for short term development. You can use the custom CSS ability to test things quickly without updating stylesheets but if you need to clear the stylesheets, you will have to use the Developer Tools to clear the cache specifically for those files.

I haven't completely figured it out but under the `Network` tab, you can turn on `Disable Cache` but you probably also have to locate the css files you loaded under `XHR`, right click, and choose `Clear Browser Cache`. The Developer Menu under View gives you the ability to reload everything so that is also handy as well as closing and reopening the whole thing.

## Over to you

If you find any problems in my code, particularly the CSS filter as I think I've missed a few images that will be inverted, make an issue or pull request.
