# V0.0.1a

### Initial Release

- 50+ Chrome automation tools via WebSocket + Chrome Extension
- Full CDP (Chrome DevTools Protocol) support: debug, network intercept, eval, snapshots
- Tab management: list, switch, new, close, group, pin, mute
- Page interaction: click, type, hover, scroll, key press, visual click (X/Y)
- Network tools: intercept, mock response, modify headers, replay request, export HAR
- Session management: save/restore cookies + localStorage
- Accessibility: full AX tree, find by label/role
- OCR: extract text with bounding box coordinates
- WebAuthn virtual authenticator for passkey/FIDO2 testing
- Tool Graph: smart execution planner for AI agents

### v0.0.2 — 2026-05-27
  - Advanced mouse: double_click, right_click, middle_click, drag_drop
  - Keyboard shortcuts: keyboard_shortcut (Ctrl+C, Ctrl+V, Ctrl+A, etc.)
  - Wait tools: wait_for_navigation, wait_for_network_idle
  - Screenshots: screenshot_element, screenshot_fullpage, pdf_print
  - CSS injection: inject_css, remove_css
  - Text selection: select_text, get_selected_text, focus_element, clear_input
  - Emulation: mock_geolocation, mock_timezone, mock_locale, mock_battery, mock_media_type, emulate_vision, cpu_throttle
  - Network & time: mock_date_time, modify_response_body, get_ws_frames, set_extra_headers, get_request_body
  - Profiling: profiling_start, profiling_stop, heap_snapshot, trace_start, trace_stop
  - Debugger control: pause_on_exception, debugger_resume, debugger_step_over, debugger_step_into, debugger_step_out, get_call_frames
  - Runtime inspection: evaluate_on_call_frame, get_script_source, live_edit_script, call_function_on, get_properties, compile_script
  - Storage: get_indexeddb, get_session_storage, get_cache_storage
  - Security: get_security_state, ignore_cert_errors
  - DOM manipulation: set_color_scheme, highlight_element, hide_element, dom_set_attribute, dom_remove_node