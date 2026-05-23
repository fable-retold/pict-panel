# Pict Panel

> **[&#9654; Read the Pict-Panel Documentation](https://stevenvelozo.github.io/pict-panel/)** &mdash; interactive docs with the full API reference.

[![Coverage Status](https://coveralls.io/repos/github/stevenvelozo/pict-panel/badge.svg?branch=main)](https://coveralls.io/github/stevenvelozo/pict-panel?branch=main)
[![Build Status](https://github.com/stevenvelozo/pict-panel/workflows/Tests/badge.svg)](https://github.com/stevenvelozo/pict-panel/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

A developer panel for inspecting and debugging Pict applications at runtime. Inject it into any running Pict app to browse AppData, templates, views, and providers -- with live template editing and persistent override management.

Open your browser console and paste:

```javascript
fetch('https://cdn.jsdelivr.net/npm/pict-panel/dist/pict-panel.js').then(r=>r.text()).then(eval).then(()=>PictPanel.inject())
```

## Features

- **Hotloadable** -- Inject into any running Pict application via a single console command
- **AppData Browser** -- Inspect and navigate the live application data tree
- **Template Browser** -- View, search, and edit templates with a built-in editor
- **View Browser** -- List all registered views with render and renderAsync actions
- **Provider Browser** -- List all registered providers with solve actions
- **Template Overrides** -- Edit templates, persist changes across reloads, toggle overrides on/off, and export as JSON or JS
- **Draggable Panel** -- Move, resize, pin, maximize, collapse to tab, and toggle dark mode
- **Persistent State** -- Panel position, size, behavior toggles, active view, and template overrides survive page reloads via localStorage

## Installation

```bash
npm install pict-panel
```

## Hotloading

Inject the panel into any page with a running Pict instance:

```javascript
fetch('https://cdn.jsdelivr.net/npm/pict-panel/dist/pict-panel.js').then(r=>r.text()).then(eval).then(()=>PictPanel.inject())
```

Or from a remote URL:

```javascript
fetch('http://localhost:9998/pict-panel.js').then(r=>r.text()).then(eval).then(()=>PictPanel.inject())
```

## Programmatic Usage

```javascript
const PictPanel = require('pict-panel');

// Register and show the panel
pict.addProvider('PictPanel', {}, PictPanel);
pict.providers.PictPanel.show();
```

## Panel Controls

The header icon grid provides nine behavior toggles:

| Toggle | Description |
|--------|-------------|
| Tab Mode | Collapse to a small logo tab at the top of the screen |
| Maximize | Expand to fill the viewport with 15px padding |
| Lock Position | Prevent dragging |
| Pin Top | Lock vertical position to top edge |
| Pin Right | Lock horizontal position to right edge |
| Night Mode | Dark color scheme |
| Resize Handle | Enable/disable the browser resize handle |
| Show UI | Hide navigation and content, leaving only the header |
| Show Navigation | Hide just the navigation bar |

## Navigation Views

| View | Description |
|------|-------------|
| AppData | Tree browser for `_Pict.AppData` with expandable nodes and inline editing |
| Templates | Searchable list of all registered templates with a built-in editor |
| Views | List of registered views with render/renderAsync action buttons |
| Providers | List of registered providers with solve action buttons |
| Overrides | Manage persisted template overrides with per-template toggle and JSON/JS export |

## Template Override System

When you edit a template in the Template Browser and save it, the original is automatically snapshotted. The Overrides view lets you:

- Toggle individual overrides on/off (restoring or re-applying the template)
- Toggle all overrides on/off at once
- Remove overrides (permanently restoring the original)
- Export active overrides as JSON or as JS source with backtick template literals

Overrides persist in localStorage and are automatically re-applied when the panel is injected after a page reload.

## Architecture

Pict Panel is built entirely with Pict's own MVC primitives:

- **Providers** -- `PP-Router`, `PP-CSS-Hotloader`, `PP-ConfigStorage`, `PP-TemplateOverrideStorage`
- **Views** -- `PP-Main`, `PP-Nav`, `PP-ADB`, `PP-TB`, `PP-VB`, `PP-PB`, `PP-TO`
- **CSS** -- Hot-injected via a concatenated style tag, no external stylesheets

## Building

```bash
npm run build
```

This uses Quackage to produce browserified bundles in `dist/`.

## Testing

```bash
npm test
npm run coverage
```

## Related Packages

- [pict](https://github.com/stevenvelozo/pict) - MVC application framework
- [pict-view](https://github.com/stevenvelozo/pict-view) - View base class
- [pict-provider](https://github.com/stevenvelozo/pict-provider) - Data provider base class
- [pict-template](https://github.com/stevenvelozo/pict-template) - Template engine
- [fable](https://github.com/stevenvelozo/fable) - Core service ecosystem

## License

MIT

## Contributing

Pull requests are welcome. For details on our code of conduct, contribution process, and testing requirements, see the [Retold Contributing Guide](https://github.com/stevenvelozo/retold/blob/main/docs/contributing.md).
