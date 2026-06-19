# PyQt6 Module Development Playbook

**Applies to:** Tag Writer, Audio Tag Writer, HST-Metadata, QR Code Generator, and all future MCOS GUI modules  
**Last updated:** 2026-06-19

---

## The Shared Module Ecosystem

All SynchroSoft PyQt6 apps draw from a set of shared modules that live as sibling directories under `~/Projects/`. These modules solve cross-platform Qt problems once; every app consumes them via a thin wrapper.

| Module | Location | Purpose |
|---|---|---|
| Icon Manager Module | `~/Projects/Icon_Manager_Module/` | Cross-platform icon loading, Windows taskbar / AppUserModelID fix |
| ThemeManager | `~/Projects/ThemeManager/` | 6-theme `QPalette` system, `QSettings` persistence |
| ZoomManager | `~/Projects/zoom-manager/` | Singleton font-scaling zoom (75%–200%), discrete levels, `QSettings` persistence, `zoom_changed` signal |
| pyqt-app-info | `~/Projects/pyqt-app-info/` | App identity dataclass, About dialog |
| version-checker-module | `~/Projects/version-checker-module/` | Update checking |

None of these are installed packages. Each app adds the module's directory to `sys.path` at module-import time using a path-setup block at the top of whatever file first needs it.

---

## Lessons Learned

### 1. Use shared modules from day one — never prototype locally

Both the QR Code app and Tag Writer started with a local `theme.py` that duplicated what ThemeManager already provides — hundreds of lines of palette dicts and CSS generation. The QR app also started with a hand-rolled `get_app_icon()` instead of using the Icon Manager Module. In both cases the migration cost (two sessions, two version bumps each) was entirely avoidable.

**Rule:** Before writing any icon, theme, About dialog, or update-check code, check the shared module ecosystem first. Write a thin wrapper; never a reimplementation.

### 2. Module initialization order is load-bearing

Four constraints are absolute, regardless of which icon strategy the app uses:

```
A. Windows App User Model ID set  ←── before QApplication()
B. QApplication(sys.argv)
C. app.setOrganizationName() / app.setApplicationName()  ←── before any QSettings read
D. MainWindow.__init__()  ←── reads theme from QSettings or config; org/app name must already be set
E. window.show()
F. Taskbar icon assigned to window  ←── after show(); requires a valid native handle (winId())
```

How each step is implemented depends on whether the app uses the Icon Manager Module or a local `platform.py`:

| Step | Using IMM (QR Code Generator) | Using local `platform.py` (TW, ATW) |
|---|---|---|
| **A** | IMM import at module level fires `_init_win32()` automatically | Explicit `set_app_user_model_id()` call at top of `main()` |
| **B–C** | same | same |
| **D** | `MainWindow.__init__()` reads `QSettings` for theme | `MainWindow.__init__()` reads JSON config for theme |
| **E** | same | same |
| **F** | `_app_icons.set_taskbar_icon(window, APP_USER_MODEL_ID)` | `set_windows_taskbar_icon(int(window.winId()))` |

App icon (`app.setWindowIcon()`): IMM apps call `_app_icons.get_app_icon()`; local apps call `QIcon(_get_icon_path())`. Both are called after step C and before `MainWindow.__init__()`.

Any deviation from the A–F ordering causes silent failures: wrong taskbar icon, QSettings reading the wrong scope, or the wrong theme on first launch.

### 3. `APP_USER_MODEL_ID` must be version-free

The Windows App User Model ID is used by the shell to group taskbar buttons and support pinned shortcuts. **Including the version number breaks taskbar pinning on every release** — Windows treats each versioned string as a different application.

Correct format: `"SynchroSoft.<AppShortName>"` — stable across all releases.

```python
# constants.py — correct
APP_USER_MODEL_ID = "SynchroSoft.QRG"        # QR Code Generator
APP_USER_MODEL_ID = "SynchroSoft.TagWriter"   # Tag Writer
APP_USER_MODEL_ID = "SynchroSoft.ATW"         # Audio Tag Writer
```

The QR Code Generator gets this right: it constructs the ID at call time from `APP_ORGANIZATION` and `APP_SHORT_NAME` so no separate constant is needed and the value is inherently version-free. Tag Writer and Audio Tag Writer currently embed the version (`"SynchroSoft.TagWriter.TW.0.2.6"`) and should be updated.

The ID belongs in `constants.py` — the `.spec` file and any future Windows installer need to reference it from one place.

### 4. `resources/icons/` is always generated, never hand-crafted

Run `generate_icons.py` from the Icon Manager Module once when the project is created. It takes a single high-resolution PNG and produces the full set: `app_16x16.png` through `app_256x256.png`, `app.png`, `app.ico`, `app.icns`. Never edit these files by hand.

The `.gitignore` always needs a negation rule alongside the standard `*.png` exclusion:

```gitignore
__pycache__/
*.png
!resources/icons/*.png
```

### 5. `theme.py` is always a thin wrapper — never a reimplementation

**Base template** (all apps):

```python
# theme.py
from __future__ import annotations
import os, sys

_TM_PATH = os.path.expanduser("~/Projects/ThemeManager")
if os.path.isdir(_TM_PATH) and _TM_PATH not in sys.path:
    sys.path.insert(0, _TM_PATH)

from theme_manager import (  # noqa: F401
    ThemeColors, UIPalette, Theme, ThemeCategory,
    get_theme_registry, get_fusion_palette, detect_system_theme,
)

DEFAULT_THEME = "light"

# Keep in sync with every dark theme registered below
_DARK_THEMES = {"dark", "solarized_dark", "dracula"}

def is_dark_theme(name: str) -> bool:
    return name in _DARK_THEMES
```

**Optional: `canvas_color()` helper** — needed only by apps with a content preview area (QR Code Generator, HST-Metadata):

```python
def canvas_color(name: str) -> str:
    theme = get_theme_registry().get_theme(name)
    return theme.ui_palette.base_color if theme else "#ffffff"
```

**Optional: custom theme registration** — when the app needs themes beyond ThemeManager's six built-ins, register them at module import time so they are available to the whole registry:

```python
def _register_app_themes() -> None:
    registry = get_theme_registry()
    registry.register_theme(Theme(
        name="warm_light",
        display_name="Warm Light",
        content_colors=ThemeColors(...),
        ui_palette=UIPalette(
            window_color="#eee9e0", window_text_color="#504741",
            base_color="#f0ebe2", alternate_base_color="#e6e1d7",
            text_color="#504741", button_color="#e6e1d7",
            button_text_color="#504741", highlight_color="#6699cc",
            highlighted_text_color="#ffffff", secondary_highlight_color="#cc6666",
        ),
        description="Warm neutral light theme",
        is_built_in=False,
        category=ThemeCategory.CUSTOM,
    ))
    # ... additional themes

_register_app_themes()
```

Add any registered dark themes to `_DARK_THEMES` — that set is the sole source of truth for `is_dark_theme()`.

**Optional: `LEGACY_NAME_MAP`** — required when migrating an app that previously stored human-readable theme names (e.g., `"Default Light"`, `"GitHub Dark"`) in its config file. Maps old display names to ThemeManager internal registry keys so existing config files are upgraded silently on first load:

```python
LEGACY_NAME_MAP: dict[str, str] = {
    "Default Light":   "light",
    "Dark":            "dark",
    "Solarized Light": "solarized_light",
    "Solarized Dark":  "solarized_dark",
    "High Contrast":   "high_contrast",
    "Monokai":         "monokai",
    "GitHub Dark":     "github_dark",
}
```

Use this map in `config.py`'s `load_config()` (Pattern B) or the QSettings load block (Pattern A). New apps starting from scratch do not need it — the registry keys are used directly from day one.

### 6. `apply_theme()` uses `QPalette`, not `setStyleSheet()`

ThemeManager's `get_fusion_palette()` returns a `QPalette` configured for Qt's Fusion style. The correct application:

```python
def apply_theme(self):
    QApplication.instance().setPalette(get_fusion_palette(self.current_theme))
    if hasattr(self, "dark_mode_action"):
        self.dark_mode_action.setChecked(is_dark_theme(self.current_theme))
```

`apply_theme()` only sets the palette. Persistence (saving the chosen theme) is the caller's responsibility — see Lesson 7.

Never call `app.setStyleSheet()` for whole-app theming — it fights the palette and produces inconsistent results on different widget types. Widget-scoped stylesheets (e.g., the scroll area canvas color, zoom font-size CSS) remain fine.

### 7. Theme persistence — use whatever config the app already owns

The right persistence mechanism depends on what the app already uses for its other settings. Do not introduce a second mechanism just for theme.

**Pattern A — QSettings** (new modules, apps with no existing config file):

Save on change:
```python
QSettings().setValue("theme/current", self.current_theme)
```
Load on startup (after `app.setOrganizationName()` / `app.setApplicationName()`):
```python
saved = QSettings().value("theme/current", DEFAULT_THEME)
self.current_theme = saved if get_theme_registry().get_theme(saved) else DEFAULT_THEME
```

**Pattern B — JSON config** (apps that already persist settings to a `~/.app_config.json`):

Save on change (via the app's existing `config.save_config()`):
```python
config.current_theme = self.current_theme
config.save_config()
```
Load on startup — apply a migration dict in `load_config()` to convert any legacy display-name keys:
```python
_THEME_MIGRATION = {
    "Default Light": "light", "Dark": "dark",
    "Solarized Light": "solarized_light", ...
}

raw_theme = data.get('current_theme', DEFAULT_THEME)
self.current_theme = _THEME_MIGRATION.get(raw_theme, raw_theme)
```

**Both patterns** need the same validation guard to protect against stale keys when the theme set changes between versions:
```python
if not get_theme_registry().get_theme(self.current_theme):
    self.current_theme = DEFAULT_THEME
```

QR Code Generator uses Pattern A. Tag Writer and Audio Tag Writer use Pattern B.

### 8. PySide6 compatibility is already handled

MCOS targets PySide6; the sibling standalone apps use PyQt6. Both ThemeManager and the Icon Manager Module detect the available framework at import time (`_QT_FRAMEWORK` flag). The thin wrappers in `theme.py` and `main.py` need no changes. Only the `from PyQt6...` imports in the mixin files need updating when porting a standalone module into the MCOS desktop client.

---

## Standard Project Structure

```
<module>/
  <module>_gui.py              # thin launcher: sys.path setup + call main()
  ICON_<module>.png            # source icon (512×512+), checked into git
  resources/
    icons/
      app.ico                  # generated — Windows multi-resolution
      app.icns                 # generated — macOS
      app.png                  # generated — Linux default
      app_16x16.png            # generated — full resolution set
      app_24x24.png
      app_32x32.png
      app_48x48.png
      app_64x64.png
      app_128x128.png
      app_256x256.png
  src/
    <module>_gui/
      __init__.py
      constants.py             # APP_NAME, APP_SHORT_NAME, APP_VERSION,
                               # APP_TIMESTAMP, APP_ORGANIZATION, APP_USER_MODEL_ID
      main.py                  # MainWindow + main(); IMM + QSettings wiring
      menu.py                  # MenuMixin
      file_ops.py              # FileOpsMixin
      view.py                  # ViewMixin (zoom + theme)
      help.py                  # HelpMixin (README, Changelog, About)
      theme.py                 # thin ThemeManager wrapper
      <domain>.py              # app-specific, Qt-free business logic
      dialogs/
        __init__.py
        theme_dialog.py        # ThemeDialog — category-grouped combo + palette swatch preview
  tests/
  CHANGELOG.md
  .gitignore
```

---

## Day-0 Checklist for a New Module

Work through this list in order. The goal is a window that opens with the correct icon and default theme before any domain logic is written.

- [ ] **Create source icon** — `ICON_<module>.png`, 512×512 or larger
- [ ] **Generate icon set**
  ```
  python ~/Projects/Icon_Manager_Module/generate_icons.py ICON_<module>.png
  ```
- [ ] **Fill `constants.py`** — all six identity fields; set `APP_USER_MODEL_ID = "SynchroSoft.<AppShortName>"`
- [ ] **Copy `theme.py` wrapper** — base template is identical across modules; add custom theme registration and `LEGACY_NAME_MAP` only if needed (see Lesson 5)
- [ ] **Copy `dialogs/theme_dialog.py`** — the implementation is the same for every app; only the module path import block at the top changes
- [ ] **Scaffold `main.py`**
  - IMM path setup + `IconLoader` at module level
  - ThemeManager import (via `theme.py`)
  - `QSettings` load in `MainWindow.__init__()`
  - `set_taskbar_icon()` after `window.show()`
- [ ] **Scaffold `help.py`** — update `AppIdentity` fields and features list; rest is boilerplate
- [ ] **Initial commit** — project compiles, window opens, icon and default theme are correct
- [ ] **Add `.gitignore`** — `__pycache__/`, `*.png`, `!resources/icons/*.png`

---

## Per-Module Development Cycle

1. Implement domain logic in `<domain>.py` (keep Qt-free where possible)
2. Add UI in the appropriate mixin file
3. Wire menu actions in `menu.py`
4. **Verify icon** — taskbar, title bar, Alt-Tab switcher, About dialog
5. **Verify all registered themes** — cycle through each in the ThemeDialog; confirm the palette swatch preview updates; confirm dark-mode toggle; restart and confirm persistence
6. Bump `APP_VERSION` and `APP_TIMESTAMP` in `constants.py` and launcher header comment
7. Update `CHANGELOG.md` before committing the bump

---

## Integrating a Standalone Module into MCOS

1. **Framework swap** — change `from PyQt6...` to `from PySide6...` in all mixin files; `theme.py` and IMM wrappers are already framework-agnostic
2. **QSettings namespace** — prefix keys with the module name to avoid collisions with other MCOS modules: `"qr_module/theme/current"`, `"qr_module/last_dir"`, etc.
3. **Entry point** — expose `main()` (or a `create_widget()` factory) so MCOS can embed the module as a tab or launch it as a managed subprocess
4. **Icon** — MCOS will supply the application-level icon; the module's `resources/icons/` set is still used for any module-specific dialogs or system tray entries

---

## Module Migration Status

| App | IMM | ThemeManager | ZoomManager | AUMID | pyqt-app-info |
|---|---|---|---|---|---|
| QR Code Generator | ✅ v0.2.2 | ✅ v0.2.2 | — | ✅ version-free | ✅ |
| Tag Writer | ❌ local `platform.py` | ✅ v0.2.7 | ❌ local CSS | ✅ v0.2.7 | — |
| Audio Tag Writer | ❌ local `platform.py` | ✅ v0.7.11 | ❌ local CSS | ✅ v0.7.11 | — |
| HST-Metadata | — | — | — | — | — |
