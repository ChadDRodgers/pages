
# Home Network Installation & Documentation Guide

This document outlines the architecture and configuration steps for your home network. It allows for a distant fiber router by using a single long ethernet run to a central 8-port switch (for TVs and primary devices), and utilizes your existing 4+1 switch to expand the single output of your secondary mesh router for IoT. The Secondary mesh router could be used to extend the Fiber Provided Router (if desired).

------------------------------
## Network Architecture Diagram

                  [ FIBER INTERNET ]
                          │
                          ▼ (Fiber Cable)
           ┌──────────────────────────────┐
           │   Fiber Provider Router      │◄─── (Main Wi-Fi SSID)
           │    (Built-in Router/DHCP)    │
           └──────────────┬───────────────┘
                          │
                          ▼ (Single Long CAT5e/CAT6 Cable)
           ┌─────────────────────────────────┐
           │        8-Port Switch            │
           └─┬──────┬──────┬──────┬─────────┬┘
             │      │      │      │         │
             ▼      ▼      ▼      ▼         ▼
          [TV 1] [TV 2] [Device] [Device] [ Mesh Router ] (Secondary SSID)
                                          └───────┬───────┘
                                                  │ (Single LAN Output)
                                                  ▼
                                      ┌───────────────────────┐
                                      │    4+1 Port Switch    │
                                      └───────┬───────────┬───┘
                                              │           │
                                              ▼           ▼
                                         [Mesh Sat1] [Mesh Sat2]

------------------------------
## Bill of Materials (Hardware Needed)

   1. Fiber Provider Router: (Existing) Located at the internet entry point.
   2. 8-Port Gigabit Switch: (Required purchase) Positioned centrally near your media center/devices.
   3. 4+1 Port Switch: (Existing) Positioned next to your Mesh Router.
   4. 1x Long CAT5e/CAT6 Cable: To bridge the distance between the fiber router and the 8-port switch.
   5. Mesh Wi-Fi System: 1 Base Router (with 1 LAN port) and 2 Satellites supporting ethernet backhaul.
   6. Short CAT5e/CAT6 Patch Cables: To interconnect switches, TVs, and mesh nodes.

------------------------------
## Step-by-Step Installation Instructions## Step 1: Establish the Core Backbone

   1. Locate any open LAN port on the back of your Fiber Provider Router.
   2. Connect your Single Long CAT5e/CAT6 cable to this port.
   3. Run this cable across the house to your central device location.
   4. Plug the other end of this long cable into Port 1 of your new 8-Port Switch.

## Step 2: Connect the TVs & Persistent Devices

   1. Plug an ethernet cable from TV 1 into Port 2 of the 8-port switch.
   2. Plug an ethernet cable from TV 2 into Port 3 of the 8-port switch.
   3. Plug any other permanent hardwired devices (PCs, consoles, etc.) into Ports 4, 5, and 6.
   Note: Because these devices pass directly through this switch to the fiber router, they successfully live on the original provider network.

## Step 3: Connect the Secondary Mesh Router

   1. Take the Main Mesh Router node.
   2. Plug an ethernet cable from its WAN / Internet port into Port 7 of the 8-port switch.
   3. Power on the Mesh Router and follow its mobile app setup to create your Secondary Wi-Fi SSID.

## Step 4: Expand and Wire the Mesh Satellites (Hardwired Backhaul)

   1. Plug an ethernet cable into the single LAN output port on the back of your Main Mesh Router.
   2. Plug the other end of that cable into the +1 (Uplink) port (or Port 1) of your existing 4+1 Switch.
   3. Run an ethernet cable from an open port on the 4+1 switch directly to Mesh Satellite 1.
   4. Run an ethernet cable from another open port on the 4+1 switch directly to Mesh Satellite 2.

------------------------------
## Crucial Configuration Tips

* Isolation of the Mesh Traffic: By placing the 4+1 switch after the Mesh Router, your hardwired satellites stay perfectly isolated inside the secondary network. Do not accidently plug the satellites into the 8-port switch, or the mesh system will fail to create a stable backhaul.
* Avoid IP Conflicts (Double NAT): Because you have two cascading routers, you may encounter "Double NAT" (which can disrupt gaming or smart home ecosystems). If you face connectivity issues, open your Mesh system's mobile app and toggle its operating mode from Router Mode to AP (Access Point) Mode or Bridge Mode. This leaves your secondary Wi-Fi network active while letting the main fiber router cleanly manage IP addresses for the whole house.



