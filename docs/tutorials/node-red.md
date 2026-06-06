# Node-RED: coachproxy Wizard + Dashboard UI (Tutorial)

## Purpose
This tutorial shows how to access the coachproxy Node-RED wizard, create a Dashboard tab, add a group, and place a Dashboard UI element (using the `template` node) into that group.

## Prerequisites
- Node-RED is installed and running (typically at `http://<host>:1880`).
- The coachproxy integration or nodes are installed in Node-RED (the workspace provides a coachproxy wizard/node).
- The `node-red-dashboard` package is installed and enabled.

## Overview
1. Open the Node-RED editor.
2. Launch the coachproxy wizard to configure coach connection/settings.
3. Add a Dashboard tab.
4. Add a Dashboard group inside that tab.
5. Add a `template` dashboard UI element, assign it to the group, and deploy.

## Steps

### 1) Open the Node-RED editor

- In a browser, navigate to `http://<node-red-host>:1880` (replace `<node-red-host>` with your host or IP).
- Log in if authentication is enabled.

### 2) Launch the coachproxy wizard

- In the Node-RED editor, look for the coachproxy integration. Common places to find it:
  - The right-hand sidebar: some node packages add a sidebar panel named “CoachProxy” or similar.
  - The node palette: type `coachproxy` in the palette search; a wizard node or helper node may appear.
- If a dedicated “CoachProxy Wizard” node exists, drag it into a flow and double-click it to open the configuration dialog or wizard button.
- Follow the prompts to configure connection details (coach host, ports, credentials, and any options the wizard requests). Save/apply the changes.

Notes: If your installation exposes a separate web-based wizard for coachproxy (for example `/coachproxy-wizard`), open that URL in your browser and follow the same configuration steps.

### 3) Add a Dashboard Tab

- Ensure the `node-red-dashboard` sidebar is visible. If not, install/enable `node-red-dashboard`.
- In the Dashboard sidebar, click the menu to add a new Tab.
- Provide a name (for example, `CoachProxy`) and optional icon.

### 4) Add a Group to the Tab

- In the Dashboard sidebar, under the new Tab, click to add a Group.
- Configure the Group name (for example, `Controls`), set width/columns and order as needed.

### 5) Add a Dashboard UI `template` element and assign it to the group

- In the Node-RED palette find the `template` node (it lives under the `dashboard` category when `node-red-dashboard` is installed).
- Drag the `template` node into your flow.
- Double-click the `template` node and set these properties:
  - **Name:** friendly name like `Coach UI Template`.
  - **Group:** select the group you created in step 4 (e.g., `Controls / CoachProxy`).
  - **Template:** paste HTML/Angular markup for the element (example below).
  - **Size:** choose appropriate width/height if needed.
- Connect the `template` node to any input nodes you need (e.g., an `inject` or a function that sends payloads) and click `Deploy`.

Example `template` content (simple card showing a value):

```html
<div style="padding:6px">
  <md-card>
    <md-card-title>Coach Status</md-card-title>
    <md-card-content>
      <div ng-bind="msg.payload.status || 'unknown'"></div>
    </md-card-content>
  </md-card>
</div>
```

Example flow snippet (conceptual):
- `inject` node (or incoming msg from coachproxy) -> `function` (map data to msg.payload) -> `template` (Dashboard group)

### 6) Test and iterate

- After deploying, open the Dashboard UI (usually `http://<node-red-host>:1880/ui`) to view the new Tab and Group.
- Trigger an inject or ensure coachproxy sends data to the flow so the template displays live values.

## Troubleshooting
- If you do not see the coachproxy wizard or nodes, verify the coachproxy node package is installed under the Node-RED palette manager.
- If the Dashboard group is not available in the `template` node config, restart Node-RED to refresh the UI list and ensure `node-red-dashboard` is active.
- Check browser console and Node-RED logs for errors if the template content does not render.

## Tips
- Use small, focused template elements for easier layout control.
- Prefer binding to `msg.payload` for simple data updates; use Angular expressions for more advanced dynamic content.

---
Created as a concise how-to for adding coachproxy-driven Dashboard UI elements using Node-RED.
