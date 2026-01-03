# Plugin Development Guide

## Guide Overview
1. **[Plugin Directory Structure](#1-plugin-directory-structure)**: Essential folder & file naming rules.
2. **[Basic Plugin Skeleton](#2-basic-plugin-skeleton)**: The copy-paste starter code (Start here!).
3. **[Action Settings UI](#3-action-settings-ui)**: Adding user options (Dropdowns, Inputs) - Automatic or Manual.
4. **[Async Settings Data](#4-async-settings-data)**: How to load data without freezing the app.
5. **[Global Plugin Configuration](#5-global-plugin-configuration)**: Creating a general settings page for your plugin.
6. **[Advanced Controls](#6-advanced-controls)**: Changing button icons/text dynamically code.
7. **[Advanced Features](#7-advanced-features)**: Pro patterns: State Sync, Hidden Metadata, Live Preview.
8. **[Packaging & Publishing](#8-packaging--publishing)**: Ready to share? Zip format and Store submission.

## 1. Plugin Directory Structure

To create a new plugin, you must create a directory within the main Plugins folder. The folder name and the main Python file must match exactly.

**Directory Path:**  
`%appdata%/Ignite Panel/Plugins/`

**Icons location:**  
All icons (plugin icon and action icons) must be placed in:  
`%appdata%/Ignite Panel/icons/`

### Structure Requirements:
1.  **Folder Name:** The name of your plugin (e.g., `MyPlugin`).
2.  **Main Python File:** Must be named exactly as the folder (e.g., `MyPlugin.py`).
3.  **Requirements File:** `requirements.txt` for any external Python dependencies.

### File Tree Example:
```text
Ignite Panel/
├── Plugins/
│   └── MyPlugin/
│       ├── MyPlugin.py       <-- Main logic (Must match folder name)
│       └── requirements.txt  <-- Dependencies (Required for libraries)
└── icons/                    <-- Place your .svg or .png icons here
```

---

## 2. Basic Plugin Skeleton

Below is a standard boilerplate for a plugin. This structure provides a clean slate for your own logic with two example actions.

Copy this code into your `MyPlugin.py` file.

```python
# MyPlugin.py
# Template for Ignite Panel Plugin

# Import libraries (add to requirements.txt if external)
# import keyboard

# ──────────────────────────────────────────────
# ✅ Plugin Metadata (ADDON_INFO)
# ──────────────────────────────────────────────
# This dictionary defines how your plugin appears in the Ignite Panel.
ADDON_INFO = {
    "name": "MyPlugin",                  # Display Name
    "description": "Description of what the plugin does",
    "version": "1.0.0",
    "author": "Your Name",
    "plugin_icon": "my_plugin_icon.svg", # Filename only (File must be in %appdata%/Ignite Panel/icons)
    "actions": [
        {
            "name": "Action 1",          # Name of the first action
            "icon": "action1_icon.svg",  # Filename only (File must be in %appdata%/Ignite Panel/icons)
            "description": "Description for action 1"
        },
        {
            "name": "Action 2",          # Name of the second action
            "icon": "action2_icon.svg",  # Filename only (File must be in %appdata%/Ignite Panel/icons)
            "description": "Description for action 2"
        }
    ]
}

# ──────────────────────────────────────────────
# ✅ Plugin Class
# ──────────────────────────────────────────────
class MyPlugin:
    """
    Main Plugin Class.
    Must be instantiated as 'plugin_instance' at the end of the file.
    """

    def execute_action(self, action_name, action_data):
        """
        Called when the user triggers an action.
        """
        try:
            if action_name == "Action 1":
                # Logic for Action 1
                print("[MyPlugin] Executing Action 1...")
                return True
            
            elif action_name == "Action 2":
                # Logic for Action 2
                print("[MyPlugin] Executing Action 2...")
                return True
            
            else:
                print(f"[MyPlugin] ❌ Unknown action: {action_name}")
                return False

        except Exception as e:
            print(f"[MyPlugin] ⚠️ Error executing action {action_name}: {e}")
            return False

    def create_settings_widget(self, action_name, saved_values, parent=None):
        """
        (Optional) Create a UI widget for action configuration.
        Return None if no settings are needed.
        """
        return None

# ──────────────────────────────────────────────
# ✅ Export Instance
# ──────────────────────────────────────────────
# This exact instance name is required for the plugin loader.
plugin_instance = MyPlugin()
```

---

## 3. Action Settings UI

If your action requires user customization (e.g., selecting an item or entering text), you have two options:

### Option A: Automatic UI (Recommended)
Define settings directly in `ADDON_INFO`. The system will generate the UI for you.

```python
"actions": [
    {
        "name": "Switch Scene",
        "type": "OBS",
        "icon": "scene.svg",
        "description": "Switch to a specific scene",
        "settings": [
            {
                "type": "select",        # 'select' for dropdown or 'text' for input
                "name": "scene_name",    # Key to retrieve value later
                "label": "Select Scene", # Label shown to user
                "options": ["Scene 1", "Scene 2", "Scene 3"],
                "required": True
            }
        ]
    }
]
```
*Accessing the value:* The selected value will be passed in `action_data` during execution:
```python
scene = action_data.get('scene_name')
```

### Option B: Manual UI (Advanced)
Use `PyQt6` to build a custom UI for complete control.

**Golden Rule:** The settings container must have a maximum width of **300px**.  
**Auto-Save:** Add the property `setting_key` to any input to handle saving automatically.

```python
from PyQt6 import QtWidgets

def create_settings_widget(self, action_name, saved_values, parent=None):
    # 1. Main Container
    w = QtWidgets.QWidget(parent)
    layout = QtWidgets.QVBoxLayout(w)

    # 2. Card Container (Fixed Width)
    card = QtWidgets.QWidget()
    card.setMaximumWidth(300)
    layout.addWidget(card)

    # 3. Input Field
    inp = QtWidgets.QLineEdit()
    inp.setPlaceholderText("Enter value...")
    
    # 4. Auto-Save Logic
    inp.setProperty("setting_key", "my_setting_id")

    # 5. Load Previous Value
    if "my_setting_id" in saved_values:
        inp.setText(saved_values["my_setting_id"])

    layout.addWidget(inp)
    return w
```

---

## 4. Async Settings Data

**Important:** Do not run heavy tasks (like API calls) inside `create_settings_widget`, or the app will freeze. Use `prepare_settings_data` to run code in the background.

### Background Logic
```python
def prepare_settings_data(self, action_name, saved_values):
    """Runs in a BACKGROUND thread."""
    if action_name == "My Heavy Action":
        # Example: Fetch data from API
        return {
            "data_list": ["Item 1", "Item 2", "Item 3"]
        }
    return None
```

### UI Logic
```python
def create_settings_widget(self, action_name, saved_values, parent=None):
    """Runs in the MAIN thread."""
    # Access data returned from background
    items = saved_values.get("data_list", [])

    w = QtWidgets.QWidget(parent)
    layout = QtWidgets.QVBoxLayout(w)

    combo = QtWidgets.QComboBox()
    combo.addItems(items)

    layout.addWidget(combo)
    return w
```

---

## 5. Global Plugin Configuration

If your plugin needs a global setup page (e.g., "Link Discord Account"), create a `config.py` file in your plugin folder.

### Config UI Structure
```python
from PyQt6 import QtWidgets
import os
import json

def create_config_ui(parent=None):
    w = QtWidgets.QWidget(parent)
    w.setMinimumSize(900, 500)

    layout = QtWidgets.QVBoxLayout(w)
    label = QtWidgets.QLabel("Plugin Configuration")
    layout.addWidget(label)

    return w
```

### Save/Load Helpers
```python
def get_config_file_path():
    path = os.path.join(
        os.path.expanduser("~"), 
        "Documents", 
        "Ignite Panel", 
        "plugin_config"
    )
    os.makedirs(path, exist_ok=True)
    return os.path.join(path, "myplugin_config.json")

def save_data(data_dict):
    with open(get_config_file_path(), "w") as f:
        json.dump(data_dict, f, indent=2)
```

---

## 6. Advanced Controls

Control the button appearance dynamically (Change Icon, Update Text) using the internal API.

```python
from Functions.button_icon_manager import (
    get_current_button, 
    change_button_icon, 
    set_toggle_icons, 
    set_toggle_state, 
    set_button_name
)

# 1. Get Current Button
btn = get_current_button()

# 2. Change Icon
if btn:
    change_button_icon(btn, r"IgnitePanel\_internal\icons\new_icon.svg")

# 3. Set Toggle Icons (On/Off state)
set_toggle_icons(
    btn, 
    r"path\icon_on.svg", 
    r"path\icon_off.svg"
)

# Update state (True = Off Icon, False = On Icon)
set_toggle_state(btn, True)

# 4. Change Text
set_button_name(btn, "Live")
```

---

---

## 7. Advanced Features

### 1. Button State Synchronization (Feedback Loop)
For "Smart" plugins (like toggle mute, play/pause), the button icon should update automatically when the app state changes (even if changed externally).

**Goal:**  
- Register the button when it's created.
- Check the state periodically (e.g., in a background thread).
- Update the icon if the state changes.

#### A. Storing Button References
Store the button when `execute_action` is called or when the action is initialized.

```python
# In your __init__
self._mute_buttons = []

# Inside execute_action or registration logic
if action_name == "Mute":
    # Store button reference for future updates
    button = action_data.get('_button_ref')
    if button and button not in self._mute_buttons:
        self._mute_buttons.append(button)
```

#### B. Updating Icons Dynamically
Use the internal `button_icon_manager` to update icons safely.

```python
from Functions.button_icon_manager import set_toggle_state

def _update_mute_buttons(self, is_muted):
    # Iterate over stored buttons
    for button in self._mute_buttons[:]:
        try:
            # Check if button still exists
            if hasattr(button, '_is_toggle_mode'):
                # True = Show "On" State (e.g., Muted icon)
                # False = Show "Off" State (e.g., Unmuted icon)
                set_toggle_state(button, is_muted)
        except RuntimeError:
            # Button was deleted
            self._mute_buttons.remove(button)
```

### 2. Hidden Metadata Storage
Sometimes you need to store data with the button that the user doesn't see (like a Window ID or Process Path).

**Technique:** Use a hidden `QLineEdit` in your Settings UI.

```python
# In create_settings_widget (Manual UI)
def create_settings_widget(self, ...):
    # ... setup layout ...

    # Create hidden input
    hidden_input = QtWidgets.QLineEdit()
    hidden_input.setVisible(False)
    hidden_input.setProperty('setting_key', 'window_hwnd') # Auto-save key
    
    # Set value programmatically
    hidden_input.setText("123456")
    
    layout.addWidget(hidden_input)
    return w
```
The value "123456" will now be saved in `saved_values['window_hwnd']` and available in `execute_action`.

### 3. Setup Toggle Icons (Code-First)
Instead of setting icons in the JSON or Editor, you can enforce specific toggle icons from code.

```python
from Functions.button_icon_manager import set_toggle_icons
from path_utils import resource_path
import os

def _setup_icons(self, button):
    icon_on = resource_path(os.path.join("icons", "mic_muted.svg"))
    icon_off = resource_path(os.path.join("icons", "mic_active.svg"))
    
    # Register the two states for this button
    set_toggle_icons(button, icon_on, icon_off)
    # Set initial state (False = Main Icon)
    set_toggle_state(button, False) 
```

### 4. Editor Live Preview (UX Magic)
Make your Settings UI interact with the button immediately. For example, changing the button icon *while* the user selects an option in the menu.

**How to get the current button from Settings UI:**

```python
from Functions.button_icon_manager import get_current_button, change_button_icon

# Inside create_settings_widget
def create_settings_widget(self, action_name, saved_values, parent=None):
    # ... create combo box ...
    
    def on_change():
        # Get the button currently being edited
        btn = get_current_button()
        if btn:
             # Example: Change icon based on selection
             new_icon_path = r"C:\path\to\icon.png"
             change_button_icon(btn, new_icon_path)
             
    combo.currentIndexChanged.connect(on_change)
    return w
```

### 5. Async UI Loading (No Freeze)
If you need to load hundreds of items (like Steam Games) into a combo box *after* the UI opens without freezing the app.

**Pattern:** Use `ThreadPoolExecutor` inside `create_settings_widget`.

```python
import threading
from concurrent.futures import ThreadPoolExecutor

def create_settings_widget(self, action_name, saved_values, parent=None):
    # 1. Create Empty Combo
    combo = QtWidgets.QComboBox()
    combo.addItem("Loading...")
    
    # 2. Define Background Task
    def load_data():
        # simulate heavy work
        items = ["Game A", "Game B", "Game C"] 
        return items

    # 3. Define Callback (Runs on Main Thread)
    def on_loaded(future):
        items = future.result()
        combo.clear()
        combo.addItems(items)

    # 4. Start Thread
    def start_loading():
        executor = ThreadPoolExecutor(max_workers=1)
        future = executor.submit(load_data)
        # Use a timer to check result to keep it safe for UI
        # (Simplified for guide; use QThread/Signals in production for best safety)
        pass 
        
    # Ideally use QThread, but Python threads work for simple cases if careful.
    threading.Thread(target=lambda: [combo.addItems(load_data())], daemon=True).start()
    
    return w
```

---

## 8. Packaging & Publishing

Follow these steps to publish your plugin to the Store.

### 1. Create Manifest
Create a `manifest.json` file in your GitHub repo root.  
**Important:** The icon must be a direct URL to the image file.

```json
{
  "name": "Media",
  "version": "1.0.0",
  "description": "Control your media playback.",
  "author": "YourName",
  "icon": "https://github.com/User/Repo/blob/main/Media/media.svg" 
}
```

### 2. Zip Structure
You must create a root folder named `Plugins` before zipping.

```text
Plugins/               <-- Zip this folder!
├── Media/             <-- Your code
│   └── Media.py
└── icons/             <-- Your SVG/PNG icons
    └── play.svg
```

### 3. Create Release
1.  Create a GitHub Release.
2.  **Tag** must match your manifest version (e.g., `1.0.0`).
3.  Upload the `.zip` file as an asset.

### 4. Submit to Store
1.  Go to the developer dashboard: [https://ignitepanel.pages.dev](https://ignitepanel.pages.dev)
2.  Click **Upload**.
3.  Paste your **GitHub Repository URL**.

> **Note:** For Mono-Repos (multiple plugins in one repo), paste the URL to the specific plugin folder (e.g., `github.com/User/Repo/tree/main/MyPlugin`) instead of the root URL.
