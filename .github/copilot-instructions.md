<!-- Copilot/AI agent instructions for node-red-contrib-nibepi -->
# Quick Guide for AI coding agents

Purpose: help an AI quickly become productive in this repository (Node-RED nodes that integrate with Nibe F-series heatpumps via the `nibepi` core).

1) Big picture
- Code implements a set of Node-RED custom nodes (prefixed `nibe-`) that communicate with the `nibepi` core module to read/write registers on Nibe heatpumps.
- Each Node-RED node is implemented as a pair: an editor UI (`*.html`) and runtime logic (`*.js`). Mapping is in `package.json` under `node-red.nodes` (e.g. `"nibe-input": "input.js"`).
- `config_node.js` is the central config/plugin manager. It initializes `nibepi`, exposes an event emitter `server.nibeData` and manages register lists, plugin initialization and MQTT integration.

2) Runtime / data flow
- `nibepi` (dependency in `package.json`) handles serial/TCP communication and provides async helpers (see `config_node.js` usages: `nibe.reqData`, `nibe.setData`, `nibe.initiateCore`).
- Nodes subscribe to events from the config node via `server.nibeData.on('...')`. Common events: `'data'`, register-specific events (register number or name), `'fault'`, `'ready'`, `'config'`.
- Message conventions: nodes commonly send objects like `{ topic: <register>, payload: <value>, raw: <meta> }`. Example: `nibe-input` sends two outputs — first only when value changed, second always (see `input.js`).

3) Key files to inspect (start here)
- `package.json` — node mapping and dependencies.
- `config_node.js` — core glue: nibepi init, `nibeData` emitter, plugin/register management (getList/initiatePlugin), MQTT integration.
- `input.js`, `output.js`, `request.js` — representative node runtime patterns and message shapes. See `input.js` for the two-output pattern and `node.status` usage.
- `*.html` — editor UI definitions for each node (controls saved in node's `config` object).
- `language-EN.json`, `language-SE.json`, `translate.json` — UI text/translation usage.

4) Project-specific conventions and patterns
- Node naming: all nodes use the `nibe-` prefix and are registered via `RED.nodes.registerType("nibe-<name>", ...)`.
- Config node exposes `server.nibe` (API wrapper) and `server.nibeData` (EventEmitter). Nodes access them using `RED.nodes.getNode(config.server)` and then `server.nibeData.on(...)`.
- Registers: many files reference register names via `server.hP()` lookup — the config node maintains register -> numeric mapping. When authoring a node, prefer using `server.hP()[config.name]` when available.
- Two-output pattern: some nodes return [changedOnly, always] outputs. Follow `input.js` pattern when adding similar behavior.

5) Developer workflows (how to run / test changes locally)
- This repo is a Node-RED package. To test locally from Windows PowerShell:
  - In the repo: `cd \path\to\node-red-contrib-nibepi`
  - Create a link and install into your Node-RED user dir:
    ```powershell
    npm link
    cd $env:USERPROFILE\.node-red
    npm link node-red-contrib-nibepi
    # restart Node-RED to load the new node
    ```
  - Alternatively pack and install: `npm pack` and then `npm install <tgz>` inside your Node-RED folder.
- `package.json` has no tests; `npm start` would run `index.js` but `index.js` is not present — do not rely on `npm start` for Node-RED testing. Run Node-RED and view its log for node errors.

6) Debugging tips
- Node-RED logs and the Node-RED editor sidebar are the primary debug surfaces. Nodes use `node.status(...)` and emit messages to help diagnose runtime state (see `input.js`).
- `config_node.js` extensively uses console logging and `nibeData.emit(...)` — inspect emitted event names during runtime when investigating missing events.

7) Integration points & external dependencies
- `nibepi` (github:anerdins/nibepi#master) — core serial/TCP logic. Changes to protocol-level behavior live there; prefer adjusting `nibepi` for transport/serialization fixes.
- Optional MQTT: config node supports `mqtt` and `mqtt_discovery` flags and publishes to the configured `mqtt_topic` (default `nibe/modbus/`).
- UI/dashboard integration: the package depends on `node-red-dashboard` and `node-red-node-ui-list` and provides some dashboard UI in `*.html`/`*-dashboard` code.

8) When changing code — concrete checklist for AI edits
- Keep node API stable: nodes expect `server.nibe` and `server.nibeData` to exist on the config node.
- Update both `.js` runtime and `.html` editor files together when changing configuration properties.
- Follow the two-output conventions and `topic`/`payload` shapes used in `input.js` and the README example.
- If you add/rename nodes, update `package.json` `node-red.nodes` mapping.

9) What this file does NOT cover
- Internal implementation details of `nibepi` (refer to the `nibepi` repo for transport-level logic).
- Non-discoverable developer processes (CI or private deploy scripts) — only repo-observable workflows are included.

If anything is unclear or you'd like more detail about a specific node or flow (for example `request.js` behavior or MQTT publishing calls in `config_node.js`), tell me which files to expand and I'll update this file.
