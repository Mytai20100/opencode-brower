# Opencode Browser Agent — SKILL.md

## Non-Negotiable Rules

1. **Call `chrome_get_tool_graph` FIRST** before any complex task — returns `execution_order` and `avoid_these`, costs <50 tokens.
2. **Never re-call** a tool whose result is already available (tabId, selector, requestId — reuse directly).
3. **Prefer low cost first**: LOW -> MEDIUM -> HIGH.
4. **Never close a user's tab** unless explicitly requested.
5. **Never call "defensive" tools** (screenshot, get_html, snapshot) unless visual inspection is genuinely required.

---

## Cost Reference

| Cost | Tools |
|:---:|:---|
| **LOW** | `get_active_tab` `list_tabs` `search_tabs` `get_page_info` `get_content` `get_workflow_context` `navigate` `click` `type` `key_press` `get_cookies` `get_local_storage` `get_session_storage` `get_history` `debug_get_logs` `debug_get_network` `save_session` `restore_session` `focus_element` `clear_input` `get_selected_text` |
| **MEDIUM** | `debug_attach` `debug_eval` `find_accessible_nodes` `find_text_on_screen` `visual_click` `intercept_request` `modify_headers` `replay_request` `double_click` `right_click` `middle_click` `drag_drop` `keyboard_shortcut` `wait_for_navigation` `wait_for_network_idle` `inject_css` `remove_css` `select_text` `switch_iframe` `upload_file` `grant_permissions` `ocr_page` `debug_emulate_*` `mock_*` `cpu_throttle` `modify_response_body` `set_extra_headers` `get_request_body` `profiling_start` `profiling_stop` `trace_start` `trace_stop` `pause_on_exception` `debugger_*` `live_edit_script` `get_indexeddb` `get_cache_storage` `dom_set_attribute` `dom_remove_node` `highlight_element` `hide_element` `set_color_scheme` |
| **HIGH — avoid** | `screenshot` `screenshot_element` `screenshot_fullpage` `pdf_print` `get_html` `debug_get_dom_snapshot` `heap_snapshot` `get_accessibility_tree` |

---

## Decision Flow by Intent

### Reading page content
```
get_content          -> full visible text (1 call)
get_page_info        -> title, url, links, meta (1 call)
WRONG: get_html      -> only when raw HTML is strictly needed
WRONG: screenshot + OCR -> wasteful; use get_content instead
```

### Click / Type on page
```
1. get_workflow_context  -> get selectors for forms/buttons/inputs
2. click(selector) / type(selector, text)
WRONG: screenshot before clicking
WRONG: find_elements -> get_element_info -> click  (3 calls, redundant)
```

### Finding a hard-to-locate element
```
Try in order, stop when found:
1. get_workflow_context
2. find_accessible_nodes(label/role)
3. find_text_on_screen(query)  -> returns {x, y}
4. visual_click(x, y)          -> last resort
```

### Monitor Network / API
```
1. debug_attach
2. [perform actions on page]
3. debug_get_network(filter="/api/")
4. debug_get_response_body(requestId)   <- only if body is needed
WRONG: execute_script fetch(...) -> cannot capture background requests
```

### Reading console logs
```
1. debug_attach
2. [trigger the action that causes the error]
3. debug_get_logs(level="error")
```

### Managing multiple tabs
```
1. list_tabs                   -> get tabIds (call once, cache the result)
2. switch_tab(tabId)
   or search_tabs(query)       -> when many tabs are open
WRONG: get_active_tab + list_tabs -> duplicate info, 2 calls
```

### Cookies (including HttpOnly)
```
Standard:  get_cookies(url)
HttpOnly:
  1. debug_attach
  2. debug_get_cookies
WRONG: execute_script("document.cookie") -> HttpOnly cookies invisible
```

### Session management
```
Save:    save_session(name="myapp")
Restore:
  1. restore_session(name="myapp")
  2. reload_tab
WRONG: set_cookie one by one -> too many calls
```

### Mobile / responsive testing
```
1. debug_attach
2. debug_emulate_device(width=375, height=812)
3. screenshot   <- only when visual confirmation is needed
```

---

## Prerequisites

| Tool | Requires first |
|:---|:---|
| All `debug_*` | `debug_attach` (attach once per tab only) |
| `switch_tab` | `list_tabs` (needs tabId) |
| `mock_response` | `intercept_request` |
| `debug_get_response_body` | `debug_get_network` |
| `switch_iframe` | `list_iframes` |
| `visual_click` | `find_text_on_screen` (needs coordinates) |
| `profiling_stop` | `profiling_start` |
| `trace_stop` | `trace_start` |
| `remove_css` | `inject_css` (needs cssId) |
| `wait_for_network_idle` | `debug_attach` |

---

## Anti-patterns

| Wrong | Correct |
|:---|:---|
| `screenshot` to "see" page before doing anything | `get_workflow_context` or `get_page_info` |
| `get_html` to find a selector | `find_elements` or `get_workflow_context` |
| Calling `list_tabs` multiple times | Cache tabId from first call |
| `execute_script` to fetch an API | `debug_attach` + `intercept_request` |
| `find_elements` + `get_element_info` per element | `find_elements(limit=N)` once |
| `debug_attach` multiple times on same tab | Attach once, reuse |
| `visual_click` before trying selector-based click | Always try `click(selector)` first |
| `get_accessibility_tree` to find one element | `find_accessible_nodes(query, role)` |
| Closing a tab not mentioned in the request | Never close tabs not explicitly requested |

---

## Optimized Workflow Examples

### Login to website — 8 calls
```
1. get_tool_graph(intent="login fill form")
2. navigate(url)
3. wait_for_element("form")
4. get_workflow_context
5. type("#email", "...")
6. type("#password", "...")
7. click("button[type=submit]")
8. wait_for_element(".dashboard")
```

### Inspect API responses — 4-5 calls
```
1. get_tool_graph(intent="capture network api")
2. debug_attach
3. navigate(url) / reload_tab
4. debug_get_network(filter="/api/")
5. debug_get_response_body(requestId)  <- only if needed
```

### Click element with no clear selector — 2 calls
```
1. find_text_on_screen("Submit Order")  -> {x:245, y:380}
2. visual_click(x=245, y=380)
```

---

## New Tools — Quick Reference

### Mouse & Keyboard
```js
chrome_double_click(selector)
chrome_right_click(selector)
chrome_drag_drop(fromSelector, toSelector)
chrome_keyboard_shortcut("Ctrl+S")   // Ctrl+C, Ctrl+A, Ctrl+Z, Ctrl+V
chrome_select_text(selector, start, end)
```

### Wait
```js
chrome_wait_for_navigation(timeout=30000)     // after clicking a link
chrome_wait_for_network_idle(idleTime=500)    // requires debug_attach first
```

### Screenshot (HIGH cost — use sparingly)
```js
chrome_screenshot_element(selector, format="png")   // prefer over full page
chrome_screenshot_fullpage(format="jpeg", quality=90)
chrome_pdf_print(landscape=true, printBackground=true)
```

### Testing / Mocking
```js
chrome_mock_geolocation(latitude, longitude)
chrome_mock_timezone(timezoneId="America/New_York")
chrome_mock_date_time(timestamp, freeze=true)
chrome_modify_response_body(urlPattern, newBody)    // requires intercept_request first
chrome_cpu_throttle(rate=4)   // 4x slowdown
```

### Advanced Debugging
```js
chrome_profiling_start() -> ... -> chrome_profiling_stop()
chrome_heap_snapshot()                     // use sparingly, can be large
chrome_live_edit_script(scriptId, src)     // may fail on minified code
chrome_pause_on_exception(state="all")     // requires debugger to be paused
```

### Storage & DOM
```js
chrome_get_indexeddb(databaseName, objectStoreName)
chrome_set_color_scheme(scheme="dark")
chrome_highlight_element(selector, color="red")
chrome_hide_element(selector, hide=true)
chrome_dom_set_attribute(selector, attr, value)
chrome_dom_remove_node(selector)
```

---

## Special Notes

- **VS Code / critical tabs**: NEVER close tabs whose URL contains `vscode`, `code-server`, or `localhost:8080`.
- **Extension disconnect**: Report immediately and stop calling tools — each pending call times out after 30s.
- **Ambiguous task**: Call `get_workflow_context` first to understand the current page state before planning.
- **Mock tools persist**: Reset mocks after testing.
- **`debug_attach` once per tab only** — calling it again is silently ignored by the extension.
