# 🌍 Language Support Guide

This guide explains how to add support for a new language to the application by creating a JSON translation file.

## 1. Creating the Language File

* **Location:** The language file should be placed in the designated translations folder for the application.
* **File Name:** Create a new file with the **`.json`** extension. You can name it after the language (e.g., `ar.json`, `es.json`, `en.json`) or any other name you prefer.

---

## 2. JSON File Structure

The JSON file must contain a specific structure for language definition, followed by a dictionary of strings to be translated.

### 2.1. Language Definition

At the beginning of the file, include the following core options:

| Key | Description | Possible Values / Notes |
| :--- | :--- | :--- |
| **`name`** | The name of the language that will appear to the user within the program. | Example: `"Arabic"` |
| **`translator`** | The name of the translator (Optional). | Example: `"SHKAS"` |
| **`rtl`** | Defines the text direction (Right-to-Left). | **`true`** for RTL languages (like Arabic), and **`false`** otherwise. |

**Definition Structure Example (JSON Code Block):**

```
{
  "name": "Language Name",
  "translator": "Translator (Optional)",
  "rtl": false 
```
2.2. The Strings Section ("strings")
Following the definition section, you must add the "strings" key, which holds all the texts that need translation within the program.

Each entry in this section consists of a key-value pair:

Key: This is the original English text (this must not be changed).

Value: This is the translated text for the new language (fill this part with the translation).

3. The Complete JSON File (Template)
The following template shows the complete structure of the language file. Please fill in the Value part (the text after the colon : ) with the translation for your new language.

Important Note: You must keep the Keys (the text appearing before the colon) exactly as they are, as they represent the original strings in the application.


```JSON

{
  "name": "New Language Name",
  "translator": "Translator Name",
  "rtl": false,
  "strings": {
    "Language": "",
    "Back": "",
    "General": "",
    "Quick Board": "",
    "Update": "",
    "Backup": "",
    "Debugging": "",
    "Console": "",
    "Version": "",
    "Window Settings": "",
    "Startup Settings": "",
    "Minimize to Tray": "",
    "Minimize the application to the system tray instead of closing.": "",
    "Close to Tray": "",
    "Close the application to the system tray instead of exiting.": "",
    "Start with Windows": "",
    "Launch the application automatically when Windows starts.": "",
    "Start Minimized": "",
    "Start the application minimized to the system tray.": "",
    "Quick Board Settings": "",
    "Configure Quick Board settings to access your actions quickly and easily.": "",
    "Board Size": "",
    "Overlay Position": "",
    "Choose where the Quick Board appears": "",
    "Select the grid size for your Quick Board": "",
    "Default (Full Grid)": "",
    "        2 × 2        ": "",
    "        3 × 3        ": "",
    "        4 × 4        ": "",
    "Center (default)": "",
    "Top Right": "",
    "Top Center": "",
    "Top Left": "",
    "Middle Right": "",
    "Middle Left": "",
    "Bottom Right": "",
    "Bottom Center": "",
    "Bottom Left": "",
    "Quick Board Mode": "",
    "Specific Page": "",
    "Select the page to mirror": "",
    "Choose how pages are mirrored": "",
    "Mirror All Pages": "",
    "Mirror Specific Page": "",
    "Overlay Hotkey": "",
    "Set the global hotkey to toggle the Quick Board": "",
    "Click here to capture hotkey...": "",
    "Reset to default": "",
    "Automatically close the Quick Board after performing an action": "",
    "Auto Close": "",
    "Copy All": "",
    "Save...": "",
    "Software Update": "",
    "CURRENT VERSION": "",
    "CHECK FOR UPDATES": "",
    "UPDATE NOW": "",
    "Checking for updates...": "",
    "Downloading update...": "",
    "New update available: v{version}": "",
    "You are using the latest version.": "",
    "Download failed, please try again.": "",
    "Restarting for update in {seconds}...": "",
    "Download completed successfully!": "",
    "No release data available": "",
    "Installation failed. Please run installer manually.": "",
    "Export & Import": "",
    "Export Data": "",
    "Import Data": "",
    "Source folder not found": "",
    "Save Backup": "",
    "Ignite Panel Backup (*.ip)": "",
    "Export cancelled": "",
    "Export successful": "",
    "Export failed: {e}": "",
    "Select Backup": "",
    "Import cancelled": "",
    "Invalid backup file": "",
    "Import successful": "",
    "Import Complete": "",
    "Data imported successfully!\nApplication will restart automatically to apply changes.": "",
    "Import failed: {e}": "",
    "Please restart manually: {e}": "",
    "Multi Action": "",
    "Run": "",
    "Change Icon": "",
    "Settings": "",
    "Delete": "",
    "multi action": "",
    "Drag actions from the sidebar and drop them here": "",
    "Show": "",
    "Hide": "",
    "Exit": "",
    "Change Icon": "",
    "Icon Packs": "",
    "Browse Files": "",
    "Preview": "",
    "Primary Icon": "",
    "Toggle Icon": "",
    "Enable Toggle Mode (Two Icons)": "",
    "No Image": "",
    "Loading icons...": "",
    "No icons found in this pack": "",
    "Select Image File": "",
    "🗑️ Delete Icon": "",
    "Delete Icon": "",
    "Are you sure you want to delete this icon?\n\nFile: {file}\n\nThis action cannot be undone.": "",
    "Icon Deleted": "",
    "Icon '{file}' has been deleted successfully.": "",
    "File Not Found": "",
    "Icon file not found:\n{path}": "",
    "Delete Error": "",
    "Failed to delete icon:\n{error}": "",
    "Cancel": "",
    "Apply": "",
    "Error": "",
    "Drag & Drop Icon": "",
    "Plugins Manager": "",
    "Search plugins...": "",
    "Configuration": "",
    "No plugins available": "",
    "This plugin has no settings": "",
    "No valid settings UI found in config.py for this plugin": "",
    "Failed to load settings for this plugin": "",
    "Enable": "",
    "Disable": "",
    "plugins manager": "",
    "Button settings": "",
    "Button Name": "",
    "Hotkey Shortcut": "",
    "Click here to capture hotkey...": "",
    "Clear shortcut": "",
    "Undo": "",
    "Copy": "",
    "Paste": "",
    "Information": "",
    "This action has no settings to configure.": "",
    "Loading {action_name} settings...": "",
    "Please wait...": "",
    "Error loading {action_name} settings": "",
    "Loading timeout for {action_name}": "",
    "Plugin is taking longer than expected": "",
    "Select All": ""
  }
}
