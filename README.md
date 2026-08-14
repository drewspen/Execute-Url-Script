# Execute-Url-Script

The simplest member of this author's "Execute Url Script" family of generic [Google Tag Manager (GTM)](https://tagmanager.google.com/) tag templates (`.tpl`). This one does exactly one thing: injects an external `<script>` tag from a configurable URL — no `id` attribute, no queue array, no vendor-specific initialization. It's the base case that [Execute-Url-Script-Plus-Id](https://github.com/drewspen/Execute-Url-Script-Plus-Id) and [Execute-Url-Script-Plus-Id-And-Queue](https://github.com/drewspen/Execute-Url-Script-Plus-Id-And-Queue) build on top of.

## What problem does this solve?

Some vendor snippets are about as minimal as a script embed can be: create a script element, point it at a URL, and insert it into the page — with no ID and no accompanying global variable setup. The **Survicate** snippet is the motivating example:

```html
<script type="text/javascript">
(function(w) {
  var s = document.createElement('script');
  s.src = {{Survicate Source URL}};
  s.async = true;
  var e = document.getElementsByTagName('script')[0];
  e.parentNode.insertBefore(s, e);
})(window);
</script>
```

There's no ID to assign and no global array to populate — just a URL. This template exists to cover that simplest case with a single configurable field, rather than requiring a full Custom HTML tag (or the extra unused fields in the author's other "Execute Url Script..." templates) for something this basic.

## Repository contents

| File | Description |
|---|---|
| `Execute Url Script.tpl` | The GTM community template gallery source file. Contains the template metadata, one configurable field, minimal sandboxed JavaScript logic, required web permissions (with a pre-populated multi-vendor URL allowlist), and (empty) test scenarios. |

## How this template fits into the "Execute Url Script" family

This author has published three closely related templates, each adding one more moving part than the last:

| Template | Fields | Adds |
|---|---|---|
| **Execute-Url-Script** *(this repo)* | `scriptUrl` | Just script injection |
| [Execute-Url-Script-Plus-Id](https://github.com/drewspen/Execute-Url-Script-Plus-Id) | `scriptUrl`, `scriptId` | + sets an `id` attribute on the injected `<script>` |
| [Execute-Url-Script-Plus-Id-And-Queue](https://github.com/drewspen/Execute-Url-Script-Plus-Id-And-Queue) | `scriptUrl`, `scriptId`, `siteId`, `queueName` | + pushes a value onto a named global array before injecting the script |

Use this template specifically when the vendor snippet you're replacing has **no ID and no queue variable** — just a plain, self-inserting `<script src="...">`.

## Template fields

When you create a tag from this template in GTM, you'll configure a single field:

| Field | Type | Required | Description |
|---|---|---|---|
| `scriptUrl` | Text | Effectively required | The full URL of the script to inject. Validated against `^https\:\/\/.*\/.*$` to require an `https://` URL with at least one path segment. Note this field has no `NON_EMPTY` validator, so an empty value is technically allowed by GTM even though the tag won't do anything useful without one. |

## How it works

The template's sandboxed JavaScript (`___SANDBOXED_JS_FOR_WEB_TEMPLATE___`) is about as minimal as it gets:

1. Reads `scriptUrl` from the template field.
2. Calls `injectScript(scriptUrl, data.gtmOnSuccess, data.gtmOnFailure)` — passing GTM's own `gtmOnSuccess`/`gtmOnFailure` callbacks directly as the success/failure handlers, with no fourth (ID/queue-name) argument at all.

There is no vendor-specific logic beyond the script injection itself — no globals are read or written, no queue is created, and no `id` is set on the injected element.

### A note on the commented-out validation block

Just like its sibling templates, the source includes a validation block wrapped in `/* ... */` (i.e., currently disabled):

```javascript
/*
// Basic validation (optional but recommended)
if (!scriptUrl) {
  log('Script URL is missing. Tag not fired.');
  data.gtmOnFailure();
  return;
}
if (!scriptId) {
  log('Script ID is missing. Tag not fired.');
  data.gtmOnFailure();
  return;
}
*/
```

This code is never executed as written. Notably, it also references a `scriptId` variable that **doesn't exist anywhere in this template** — this template has no `scriptId` field at all, so this block appears to have been copy-pasted from the `Execute-Url-Script-Plus-Id` template without being adapted. If you want to re-enable validation, you should remove the `scriptId` check entirely (since it doesn't apply here) and keep only the `scriptUrl` check, ideally wrapped in an `if / else` around the `injectScript` call rather than relying on a bare top-level `return;` (see the equivalent caveat in this author's Indeed.com Conversions and Execute-Url-Script-Plus-Id templates).

## Requested permissions

GTM template permissions are declared explicitly in `___WEB_PERMISSIONS___`. This template requests:

- **`logging`** — allowed to write to the console, restricted to the `debug` environment only. (Declared for the `log`/`logToConsole` reference in the code, though the only calls to it currently live inside the commented-out validation block.)
- **`inject_script`** — allowed to inject `<script>` tags, restricted to a pre-populated allowlist of vendor domains:
  - `https://survey.survicate.com/`
  - `https://vidassets.terminus.services/`
  - `https://scripts.demandbase.com/`
  - `https://conv.indeed.com/`
  - `https://gdc.indeed.com/`
  - `https://cdn.terminus.com/`

Note this allowlist is slightly shorter than the one in `Execute-Url-Script-Plus-Id` — it omits `https://stats.webleads-tracker.com/`, which makes sense given WebLeads Tracker's snippet needs the queue-array behavior only available in `Execute-Url-Script-Plus-Id-And-Queue`.

## Getting started

### Import into Google Tag Manager

1. In your GTM container, go to **Templates** → **Tag Templates** → **New**.
2. Click the **⋮** (more actions) menu → **Import**.
3. Select the `Execute Url Script.tpl` file from this repository.
4. Save the template.

### Create a tag from the template

1. Go to **Tags** → **New**.
2. Choose **Execute Url Script** as the tag type (it will appear under your custom templates).
3. Enter the **scriptUrl** — the full `https://` URL of the vendor script you want to load. It must be from one of the pre-approved domains listed above (or a domain you've added yourself).
4. Set the appropriate trigger — typically **All Pages**, unless the vendor snippet is meant to fire on a specific event/page only.
5. Save, preview, and test the tag before publishing.

### Verify it's working

- Use GTM's **Preview/Debug** mode to confirm the tag fires with no `gtmOnFailure` calls.
- Open your browser's developer tools **Network** tab and confirm the script loads with a `200` status from the expected domain.
- Confirm the underlying vendor's functionality (e.g., a Survicate survey widget) is active as expected.

### Adding a new vendor domain

If you want to use this template to load a script from a domain not already on the allowlist:

1. Open the template in the GTM template editor.
2. Go to the **Permissions** tab.
3. Under **Inject Script**, add the new domain (e.g., `https://example-vendor.com/`).
4. Re-save and re-import the template.

## Notes

- This template was created on 6/9/2025.
- The `___TESTS___` section currently contains no automated test scenarios (`scenarios: []`). Contributions adding test coverage via GTM's template testing framework are welcome.
- If the vendor snippet you're replacing also needs an `id` attribute on the script tag or a value pushed onto a global queue array, use [Execute-Url-Script-Plus-Id](https://github.com/drewspen/Execute-Url-Script-Plus-Id) or [Execute-Url-Script-Plus-Id-And-Queue](https://github.com/drewspen/Execute-Url-Script-Plus-Id-And-Queue) instead — this template intentionally supports neither.
- This is an unofficial, community-built template and is not published or endorsed by any of the vendors whose domains appear in its permission allowlist. Always review sandboxed template code and requested permissions before importing third-party templates into a production GTM container.

## License

No license file is currently included in this repository. Check with the repository owner before reusing this template in a commercial or redistributed context.
