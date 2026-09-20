# Using Ouroboros

Open [index.html](../index.html), open the Settings tile, and enter your API key, API base URL and model. The default base URL is `https://api.openai.com/v1`; choose a model available to your account. Only the Responses API is supported.

Open the Ouroboros tile and describe an app or a change. No build step is required. Tailwind and Lucide load from CDNs, and generation requires access to the configured API.

## Views

| View | Behavior |
| --- | --- |
| App map | Apps form a compact cluster. Small tiles show only icons; zooming in reveals titles and content previews. Tap or click a tile to focus it. |
| Expanded desktop | Apps open together at their preferred sizes and can be used simultaneously. Scroll to reach the rest of the workspace. |
| Focused app | One app opens at its preferred size while the others blur. Leaving focus restores the previous view and, in the expanded desktop, its scroll position. |

- The top-left Ouroboros button returns to the current overview. From that overview, clicking it again switches between the app map and expanded desktop.
- Drag to move around the app map. Use the wheel, pinch, or zoom buttons to change its scale. Focused app content scrolls normally; use Ctrl + wheel to zoom while the pointer is over it.
- Click outside a focused app, use its return button, press Esc, or zoom out to return to the previous view. In the expanded desktop, use an app's focus button to open it individually.
- The two views share the same transition logic. Apps stay mounted, so changing views preserves their inputs and event handlers.

## App contract

The model chooses each app's icon and dimensions. The host handles placement, view transitions, focus, and window controls.

Create an element with a stable unique ID and the classes `draggable resizable`, a direct `.window-header` containing its title, and a `.ui-body` for content. Append it to `#window-layer`. The classes register the app with the host.

| Attribute | Meaning |
| --- | --- |
| `data-window-title` | Optional label; otherwise taken from the header title or element ID. |
| `data-window-icon` | A PascalCase name from the full Lucide library, such as `CalendarDays` or `Scale`. Missing or invalid names use `AppWindow`. |
| `data-width`, `data-height` | Preferred dimensions in pixels, defaulting to 480 × 520. Use `fill` for an axis that should fill the available space. |
| `data-aspect-ratio` | Optional outer-window ratio, such as `1` or `16/9`, fitted inside the preferred dimensions. |

Map previews are 320px square. Expanded and focused apps use their preferred dimensions, constrained to the viewport; use inner scrolling when needed.

```html
<section id="my-counter" class="draggable resizable"
         data-window-icon="Tally5" data-width="360" data-height="320">
  <header class="window-header"><h2>Counter</h2></header>
  <div class="ui-body"><!-- App content --></div>
</section>
```

Check `lucide.icons[name]` before referencing an icon. For icons inside app content, use `lucide.createElement(lucide.icons.IconName)`; do not embed fixed SVG paths or use emoji as icons.

Use the shared `ui-label`, `ui-field`, `ui-button` and `ui-button is-secondary` classes, along with `--accent`, `--tint` and `--muted`. Keep labels concise and omit gesture instructions. The complete model instructions are embedded in `#ouroboros-system_prompt` in [index.html](../index.html).

## Saved work

- Save app content with `ouroboros.getState(id, defaultValue)` and `ouroboros.setState(id, value)`. Values must be JSON data; the getter returns a copy. Keep the same ID when replacing an app.
- Create the UI and its listeners in the same recorded `run_js` script. Reloading replays those scripts and restores saved app data.
- Settings contains token usage, uninstall, the last 10 task versions, export, import, backup restore and reset.
- Exported HTML carries app code, saved app data and history. Import replaces the current workspace and keeps one recoverable backup. API settings and token usage stay in the browser and are excluded from exports.
- Reset returns to the file's starting snapshot and keeps connection settings. Older apps that store content outside the state API need an update before that content can travel with an export.

On the same browser origin, this experimental branch seeds its workspace from existing data when available, then saves workspace changes separately. Connection settings and token usage remain shared with the original desktop. Preview URLs containing a commit hash stay on that exact version; refreshing an older link does not load a newer build.
