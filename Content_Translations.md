# Installing the Moodle Content Translation Plugin Set

This section documents how the content translation feature was installed and configured for the Moodle prototype of the EU Railways Academy.

## Tested Environment

The content translation setup worked reliably with the following environment:

| Component | Version / Source |
|---|---|
| Operating System | Ubuntu 24 |
| Moodle | Moodle 5.0.7 |
| Content Translations | 2.0.2 |
| Atto Editor | Legacy Atto editor, 2023 version from GitHub |
| Atto Translations Plugin | `moodle-atto_translations` from GitHub |
| Tiny / Filter Translation Plugin | `moodle-filter_translations`, version `1.0.0`, MOODLE_500_STABLE branch |

## Plugin Sources

The following plugins were used:

### Moodle Version

Used Version: 5.0.7

https://github.com/moodle/moodle/tree/MOODLE_500_STABLE 

### 1. Legacy Atto Editor

Source:

https://github.com/moodlehq/moodle-editor_atto

This was needed because the translation workflow depends on the Atto editor. In newer Moodle setups, Atto may no longer be available as a core editor, so it had to be installed manually as a legacy editor plugin.

### 2. Atto Translations Plugin

Source:

https://github.com/andrewhancox/moodle-atto_translations

This plugin adds the translation hash key into Moodle content when the Atto editor loads. The hash key is important because it connects the original content with its translation.

### 3. Content Translation Filter

Source:

https://github.com/andrewhancox/moodle-filter_translations/tree/MOODLE_500_STABLE

Used version:

```text
moodle-filter_translations 1.0.0
MOODLE_500_STABLE branch
````

This is the main plugin that handles the content translation functionality.

### 4. The Content Translations Main Plugin

Used Version: 2.0.2

Source:
https://moodle.org/plugins/download.php/36776/filter_translations_moodle50_2025070700.zip 

## How the Content Translation System Works

The content translation setup depends mainly on two plugins:

1. `filter_translations`
2. `atto_translations`

The `filter_translations` plugin is the main content translation filter. It manages translated content and displays translations depending on the selected language.

The `atto_translations` plugin adds a hash key to Moodle content when the Atto editor loads. This hash key works as a connection between the original content and the translated version. Without this hash key, Moodle cannot reliably identify which translation belongs to which piece of content.

For new content, the hash key is added automatically when the content is created or edited through the Atto editor. For existing Moodle content, a command line tool may need to be executed so that old content also receives the required translation spans.

## Installation Steps

### Step 1: Download the required plugins

Download the following plugin ZIP files from GitHub:

```text
https://github.com/moodlehq/moodle-editor_atto
https://github.com/andrewhancox/moodle-atto_translations
https://github.com/andrewhancox/moodle-filter_translations/tree/MOODLE_500_STABLE
https://moodle.org/plugins/download.php/36776/filter_translations_moodle50_2025070700.zip 
```

For the filter plugin, the `MOODLE_500_STABLE` branch was used because the Moodle installation was based on Moodle 5.0.7.

### Step 2: Upload the plugins through Moodle Site administration

The plugins were installed through the Moodle web interface.

Go to:

```text
Site administration
→ Plugins
→ Install plugins
```

Then upload the ZIP files one by one.

The plugins were uploaded through Site administration rather than manually copying them through the command line.

### Step 3: Complete the Moodle plugin installation

After uploading each plugin, Moodle checks the plugin package and shows the installation/upgrade page.

Continue through the installation process until Moodle confirms that the plugins are installed successfully.

After installation, Moodle may ask to update the database. Confirm the update and continue until the installation is completed.

### Step 4: Enable the Content translations filter

After the plugins are installed, the Content translations filter must be enabled.

Go to:

```text
Site administration
→ Plugins
→ Filters
→ Manage filters
```

Then configure the Content translations filter as follows:

```text
Content translations filter: On
Apply to: Content and headings
Position: Move near the top of the filter list
```

The filter should be placed near the top because the order of filters can affect how Moodle processes and displays content.

### Step 5: Enable and configure the Atto editor

Because the translation button depends on the Atto editor, Atto must be available and enabled.

Go to:

```text
Site administration
→ Plugins
→ Text editors
→ Manage editors
```

Make sure that the Atto editor is enabled.

If several editors are available, Atto should be enabled and usable for editing Moodle content.

### Step 6: Add the translations button to the Atto toolbar

After enabling Atto, the translations button must be added to the Atto toolbar configuration.

Go to:

```text
Site administration
→ Plugins
→ Text editors
→ Atto HTML editor
→ Atto toolbar settings
```

Add the following entry to the toolbar configuration:

```text
, translations
```

This adds the translation tool to the Atto editor toolbar.

### Step 7: Configure logging for missing and stale translations

Optionally, logging can be enabled to track missing and outdated translations.

Go to:

```text
Site administration
→ Plugins
→ Filters
→ Content translations
→ Settings
→ Logging
```

Recommended settings:

```text
Log missing translations: Yes
Log stale translations: Yes
```

This helps identify content that has not been translated yet or translations that are no longer up to date.

### Step 8: Add translation permissions

Users will not see the content translation icon unless they have the correct permission.

Go to:

```text
Site administration
→ Users
→ Permissions
→ Define roles
```

Then either create a translator role or add the permission to an existing role.

Required permission:

```text
filter/translations:edittranslations
```

This permission allows the user to edit translations.

Optional permissions include:

```text
filter/translations:bulkdeletetranslations
filter/translations:deletetranslations
filter/translations:editsitedefaulttranslations
filter/translations:edittranslationhashkeys
```

For a basic setup, only `filter/translations:edittranslations` is necessary.

### Step 9: Prepare existing content for translation

For content created before the plugin installation, Moodle content may not yet contain the required translation hash spans.

In that case, the command line tool from the plugin should be executed to insert the required spans into existing content.

Example path:

```text
/var/www/moodle/filter/translations/cli/
```

or, depending on the Moodle installation path:

```text
/var/www/moodle/public/filter/translations/cli/
```

The relevant script is usually:

```text
insert_spans.php
```

This step is important because old content may lose its translation connection if it does not contain the required hash keys.

New content created after installing and configuring the plugin should receive the hash key automatically through the Atto translations plugin.

## Important Observation During the Project

During the project, the translations were visible, but the UI button for editing translations by hash ID did not appear at first. This made it difficult to identify which translation belonged to which content element.

The problem was not caused by only one plugin. It was connected to the compatibility between:

```text
Moodle version
Ubuntu version
PHP version
Atto editor availability
Content translation filter version
Atto translations plugin
```

After resetting the configuration and using a compatible setup with Ubuntu 24, Moodle 5.0.7, Content Translations 2.0.2, the legacy Atto editor, the Atto translations plugin, and the MOODLE_500_STABLE filter plugin, the content translation feature worked correctly.

## Result

After completing the installation and configuration, the Moodle site was able to:

* display translated content,
* add hash keys to new content,
* connect original content with translations,
* show the translation editing interface,
* allow authorised users to edit translations,
* identify missing and stale translations.

This made it possible to create a multilingual Moodle learning platform where course content could be translated and adjusted manually instead of depending only on automatic translation.

## Reference
(Content Translations Step by Step Guide)[https://docs.moodle.org/311/en/Content_translation_plugin_set?_gl=1*owlid3*_ga*MTMwODcwNTUxOS4xNzc4NDUxNzE5*_ga_QWYJYEY9P5*czE3Nzg0NTE3MTkkbzEkZzEkdDE3Nzg0NTE5MzAkajUkbDAkaDA.#Installation] (accessed: 23 May 2026)
