{# <!--run: tetra-cli parse % README.md --> #}

This project is an attempt to fully deploy (provision) a fresh Mozilla Firefox install, that incoporates the security hardening settings from the [https://github.com/arkenfox/user.js](Arkenfox/user.js) project.
This is for version 104.0, which is after the migration to WebExtensions API.
In particular that means:

* [x] Configure advanced settings (`about:config`) via `user.js`
* [ ] Language Packs (partially implemented) and locale setting
* [ ] Configure UI setup
* [x] Extensions
* [ ] Extensions Settings
* [ ] Search bar installation and settings
* [ ] Create a fresh profile (unsure if I want to do this)
* [x] Provision a fresh profile
* [ ] Tracking updates from the upstream Arkenfox project

Inspired by things tools such as Nix, Terraform, and Ansible, this project is trying to create fully configured Firefox at the click of one button.
Additionally, this project provides a nice CLI interface to keep up with updates from Arkenfox.

# How this differs from Arkenfox workflow

This is based on the work of [https://github.com/arkenfox/user.js](Arkenfox) project.
The workflow of the Arkenfox project is that, after git cloning the project into a directory, they provide provide you with `user.js` that has defaults that are not intended to be useable out-of-the-box.
You create any number of additional JavaScript files that will override those settings, customised to your particular needs.
Then you use the script in the Arkenfox project to combine all the JavaScript files and copy to `<firefox-profile>/user.js` (e.g. `/home/jane-doe/.mozilla/firefox/34ahf5xy.default/user.js`
Additionally, you use the script to update the provided default `user.js`.


```{| body = |}
digraph {
  A -> B
  A -> C
}
{| end |}```

{$ run "dot", "-Tsvg", body $}


# Steps moving forward

It seems not possible to fully automate extension installation.
There seem to be several disparate locations that settings are stored for addons (see next section).
Thus, targetting specific files in the profile directory is not very reliable.

I did try using the WebDriver API, i.e. using the Selenium framework to talk through the `geckodriver` proxy to programatically control a Firefox instance.
The goal would be to automate the browser to click on settings to configure each extension or to import settings.
However, I've seen competiting claims that Selenium explicitly does or does not officially support connecting to an already running instance of Firefox.

Another possible route would be to connect Selenium to the debugger.
Although I was successful at making Selenium open a new instance (as one would do for normal webscrapping or integration testing), I could not figure out how to connect it to an already running firefox instance.

# Minimal Firefox Setup for storing addon settings

The five extensions I use are:

* [addons.mozilla.org/en-GB/firefox/addon/ublock-origin](ublock-origin)
* [addons.mozilla.org/en-GB/firefox/addon/noscript](NoScript) for blocking specific domains. Although uBlock Origin offers JavaScript blocking and having less extensions means less liabilities, NoScript allows me to have fine-grain control over specific domains I want to allow.
* [addons.mozilla.org/en-GB/firefox/addon/cookie-autodelete](cookie-autodelete)
* [addons.mozilla.org/en-GB/firefox/addon/history-cleaner](history-cleaner) to clear history at startup. History is on vector for finger printing, and I cannot use Firefox's clear history on exit if I want session restore, i.e. I cannot use "Privacy & Security > History > Clear history when Firefox closes > Browser & Download History" if I want "General > Startup > Open previous windows and tabs".
* [addons.mozilla.org/en-GB/firefox/addon/videospeed](Video Speed Controller) adds controls for consuming media at increased or decreased speeds. There is no way to export settings for this extension.

XPI files are just zip files.
Just putting downloaded addons in the `extensions/` directory is not sufficient.
Addons must also be registered (enabled, update schedule, available in private browsing), and you the addon configuration must be stored somewhere.


* `extensions/` for the .xpi files, or the actual addons and language packs.
* `storage/` is used for most extensions to store the data
* `addons.json` - for registration
* `addonStartup.json.lz4`
* `extension-preferences.json` is used store which addons are available during private browser sessions (implemented as a 'container').
* `extensions.json`
* `storage`
* `storage-sync-v2.sqlite` and its sibilings are for used by NoScript to store  th
* `storage-sync-v2.sqlite-shm`
* `storage-sync-v2.sqlite-wal`
* `user.js` (alternatively `pref.js`, but `pref.js` is essentially the browser store of `about:config` settings). The two main settings of concern are:
** `extensions.webextensions.uuids` which indicate which external extensions are installed
** `ext`

I am currently unsure if UUIDs for extensions change if you update them.
If they do, then storing the `browser.uiCustomisation.state`, `extensions.json`, etc. is a bit fragile.

# Useful links

* [https://github.com/arkenfox/user.js](Arkenfox/user.js), formerly ghacks/user.js, that provides detailed documentation and bash script workflow for deploying firefox advanced settings (`about:config`) that are more privacy and security focused.
* `about:debugging` potentially for webdriver usage. Gives access to controlling through Firefox's debugger, firebug.
* `about:support` for a nice listing of all the diagnostic information, like what profile we are currently using.
* about:profiles
* about:config
* [https://github.com/alza-bitz/ansible-firefox-addon](ansible-firefox-addon) is the ansible role, aka. package, for installing addons). This is code target at before Firefox migrated from XPCOM/XUL extensions to the current WebExtensions API.
* [https://github.com/alza-bitz/ansible-firefox](ansible-firefox) is the ansible role that uses `ansible-firefox-addon` to also install user.js
* [https://askubuntu.com/questions/73474/](Installing addons from scripts) stackoverflow question. Again this is for pre-WebExtensions API. In particular
* [https://github.com/Rasukarusan/shellnium](Shellnium) is a WebDriver wrapper for bash that works via curl. It is primarily setup for Chrome, but the modifying it to work for Firefox should be simple.
* [http://github.com/mozilla/geckodriver/issues/430](Gecko driver issue)
