# Ignite Panel Development Guide

This guide focuses only on how to create a plugin: structure, essential definitions, the two settings patterns, a blank skeleton you can copy, and advanced controls for button icon/name. It is organized to be crystal‑clear for beginners.

## Essentials Only
- Create a folder under `Plugins` with your plugin name, e.g. `Plugins/YourPluginName`.
- Inside it, create a Python file with the exact same name, e.g. `Plugins/YourPluginName/YourPluginName.py`.
- Define `ADDON_INFO` at the top of the file. Use `plugin_icon` for the plugin category icon and `icon` for action icons.
- Export `plugin_instance` at the end of the file.

Minimal `ADDON_INFO` skeleton (placeholders):
```python
ADDON_INFO = {
    "name": "Your Plugin Name",
    "description": "Short description",
    "version": "1.0.0",
    "author": "Your Name",
    "plugin_icon": "Your Plugin Icon Name",
    "actions": [
        {
            "name": "Your First Action Name",
            "type": "Your Plugin Name",
            "icon": "Your Action Icon Name",
            "description": "What this action does"
        }
    ]
}
```

Icon placement rule:
- Put your icon files in `IgnitePanel\_internal\icons`.
- Use the file name in `plugin_icon` and each action `icon`.

## Action Settings Patterns
- If the action has no settings: return `None` from `create_settings_widget`.
- If the action has settings: build a PyQt6 widget and set the container card width to `300` pixels. Tag inputs with `setting_key` so changes auto‑save.

No settings (use when your action does not need configuration):
```python
def create_settings_widget(self, action_name, saved_values, parent=None):
    return None
```

With settings (fixed width 300px, auto‑save via `setting_key`):
```python
def create_settings_widget(self, action_name, saved_values, parent=None):
    w = QtWidgets.QWidget(parent)
    v = QtWidgets.QVBoxLayout(w)
    card = QtWidgets.QWidget(); card.setMaximumWidth(300)
    v.addWidget(card)
    inp = QtWidgets.QLineEdit(); inp.setProperty('setting_key', 'my_value')
    v.addWidget(inp)
    return w
```

## Blank Plugin Definition (Copy‑Ready)
```python
# Plugins/YourPluginName/YourPluginName.py
from PyQt6 import QtWidgets

ADDON_INFO = {
    "name": "Your Plugin Name",
    "description": "Short description",
    "version": "1.0.0",
    "author": "Your Name",
    "plugin_icon": "Your Plugin Icon Name",
    "actions": [
        {
            "name": "Your First Action Name",
            "type": "Your Plugin Name",
            "icon": "Your Action Icon Name",
            "description": "What this action does"
        }
    ]
}

class YourPluginName:
    def execute_action(self, action_name, action_data):
        if action_name.lower() == "your first action name":
            return True
        return False

    def create_settings_widget(self, action_name, saved_values, parent=None):
        return None

plugin_instance = YourPluginName()
```

## General Python Skeleton (Single Action, No Settings)
```python
class MyPlugin:
    def execute_action(self, action_name, action_data):
        if action_name.lower() == "your action name":
            return True
        return False

    def create_settings_widget(self, action_name, saved_values, parent=None):
        return None

plugin_instance = MyPlugin()
```

## Advanced Things
Use these techniques to react to events and update the assigned button.

- Set button icon from your plugin when something changes:
```python
from Functions.button_icon_manager import change_button_icon
from PyQt6.QtWidgets import QApplication

current_button = None
app = QApplication.instance()
if app:
    for w in app.allWidgets():
        if hasattr(w, 'plugin_settings_ui') and hasattr(w.plugin_settings_ui, 'settings_manager'):
            current_button = getattr(w.plugin_settings_ui.settings_manager, 'current_button', None)
            if current_button:
                break

icon_path = r"IgnitePanel\_internal\icons\my_icon.svg"
if current_button:
    change_button_icon(current_button, icon_path)
```

- Toggle icon example (Discord Mute pattern):
```python
from Functions.button_icon_manager import set_toggle_icons, set_toggle_state

base_icon = r"IgnitePanel\_internal\icons\discord_mic_on.svg"
toggle_icon = r"IgnitePanel\_internal\icons\discord_mic_off.svg"
set_toggle_icons(current_button, base_icon, toggle_icon)

# Update by state (True = muted icon, False = unmuted icon)
set_toggle_state(current_button, True)
```

- Change button name by selection (Steam pattern):
```python
from Functions.button_icon_manager import set_button_name, clear_button_name

# Example: when a checkbox is enabled, set the button name to the selected account
set_button_name(current_button, account_combo.currentText())
# Or clear it
clear_button_name(current_button)
```

Notes:
- `plugin_icon` is the plugin’s category icon; `icon` is each action’s icon.
- Place icons in `IgnitePanel\_internal\icons` and reference them by file name.
