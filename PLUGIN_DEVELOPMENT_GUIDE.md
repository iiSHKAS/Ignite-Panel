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

## Plugin Config Page (Optional)
If your plugin needs a dedicated configuration page in the Plugins Manager (like the Discord config), add a `config.py` file to your plugin folder: `Plugins/YourPluginName/config.py`. When this file exists, the Plugins Manager shows a configuration button for your plugin.

Recognized exports:
- Functions returning a `QWidget`: `create_config_ui`, `get_config_widget`, `get_config_ui`, `build_ui`
- Classes inheriting `QWidget`: `ConfigWidget`, `ConfigWindow`, `MainWidget`
- Variables referencing a `QWidget`: `CONFIG_WIDGET`, `WIDGET`, `widget`, `main_widget`

Minimal config skeleton (Discord‑style form):
```python
# Plugins/YourPluginName/config.py
from PyQt6 import QtWidgets, QtCore, QtGui

def create_config_ui(parent=None):
    w = QtWidgets.QWidget(parent)
    w.setMinimumSize(900, 500)  # sized and centered by the manager
    v = QtWidgets.QVBoxLayout(w)

    title = QtWidgets.QLabel("Connect Discord")
    subtitle = QtWidgets.QLabel("Link your Discord account to unlock all features.")
    v.addWidget(title)
    v.addWidget(subtitle)

    form = QtWidgets.QWidget()
    fv = QtWidgets.QVBoxLayout(form)
    client_id = QtWidgets.QLineEdit(); client_id.setPlaceholderText("Enter your Discord Application ID")
    client_secret = QtWidgets.QLineEdit(); client_secret.setPlaceholderText("Enter your Discord Client Secret")
    client_secret.setEchoMode(QtWidgets.QLineEdit.EchoMode.Password)
    connect_btn = QtWidgets.QPushButton("Connect Discord")
    fv.addWidget(client_id)
    fv.addWidget(client_secret)
    v.addWidget(form)
    v.addWidget(connect_btn)
    return w

# Optional alternative exports
get_config_widget = create_config_ui
get_config_ui = create_config_ui
build_ui = create_config_ui

class ConfigWidget(QtWidgets.QWidget):
    def __init__(self, parent=None):
        super().__init__(parent)
        layout = QtWidgets.QVBoxLayout(self)
        layout.addWidget(create_config_ui(self))
```

### Saving Config Data
Store your plugin’s configuration in the user’s Documents folder, under a dedicated app directory:
- Location: `Documents/Ignite Panel/plugin_config`
- File name: use a plugin‑specific JSON file like `yourplugin_config.json`

Example helpers for saving/loading (Discord‑style):
```python
import os, json

def get_plugin_config_path():
    return os.path.join(os.path.expanduser("~"), "Documents", "Ignite Panel", "plugin_config")

def get_config_file_path():
    return os.path.join(get_plugin_config_path(), "yourplugin_config.json")

def save_credentials(client_id, client_secret):
    try:
        cfg_dir = get_plugin_config_path()
        os.makedirs(cfg_dir, exist_ok=True)
        cfg_file = get_config_file_path()
        data = {}
        if os.path.exists(cfg_file):
            with open(cfg_file, 'r') as f:
                try:
                    data = json.load(f)
                except json.JSONDecodeError:
                    pass
        data['client_id'] = client_id
        data['client_secret'] = client_secret
        with open(cfg_file, 'w') as f:
            json.dump(data, f, indent=2)
        return True
    except Exception:
        return False

def load_credentials():
    try:
        cfg_file = get_config_file_path()
        if os.path.exists(cfg_file):
            with open(cfg_file, 'r') as f:
                try:
                    data = json.load(f)
                    return data.get('client_id', ''), data.get('client_secret', '')
                except json.JSONDecodeError:
                    return '', ''
        return '', ''
    except Exception:
        return '', ''

def save_token(token_data):
    try:
        cfg_dir = get_plugin_config_path()
        os.makedirs(cfg_dir, exist_ok=True)
        cfg_file = get_config_file_path()
        existing = {}
        if os.path.exists(cfg_file):
            with open(cfg_file, 'r') as f:
                try:
                    existing = json.load(f)
                except json.JSONDecodeError:
                    pass
        existing.update(token_data)
        with open(cfg_file, 'w') as f:
            json.dump(existing, f, indent=2)
        return True
    except Exception:
        return False

def load_token():
    try:
        cfg_file = get_config_file_path()
        if os.path.exists(cfg_file):
            with open(cfg_file, 'r') as f:
                try:
                    return json.load(f)
                except json.JSONDecodeError:
                    return None
        return None
    except Exception:
        return None

def remove_token():
    try:
        cfg_file = get_config_file_path()
        if os.path.exists(cfg_file):
            os.remove(cfg_file)
            return True
        return False
    except Exception:
        return False
```

Sizing and centering: if your widget sets a fixed size (min == max), the manager removes the max constraint to center it cleanly inside the page container.

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
