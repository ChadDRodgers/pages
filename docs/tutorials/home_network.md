## Home Network Installation & Documentation Guide
This document outlines the architecture and configuration steps for your home network. It allows for distant fiber router by using a single long ethernet run to a central 8-port switch, which handles TVs, persistent hardwired devices, and a dedicated secondary mesh Wi-Fi network for IoT devices.
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
           ┌──────────────────────────────┐
           │        8-Port Switch         │
           └─┬──────┬──────┬──────┬──────┬┘
             │      │      │      │      │
             ▼      ▼      ▼      ▼      ▼
          [TV 1] [TV 2] [Device] [Device] [ Mesh Router ] (Secondary SSID)
                                          └───────┬───────┘
                                                  │
                                            ┌─────┴─────┐
                                            │           │
                                            ▼ (LAN)     ▼ (LAN)
                                         [Mesh Sat1] [Mesh Sat2]

------------------------------
## Bill of Materials (Hardware Needed)

   1. Fiber Provider Router: (Existing) Located at the internet entry point.
   2. 8-Port Gigabit Switch: (Required purchase) Positioned centrally near your media center/devices.
   3. 1x Long CAT5e/CAT6 Cable: To bridge the distance between the fiber router and the 8-port switch.
   4. Mesh Wi-Fi System: 1 Base Router and 2 Satellites supporting ethernet backhaul.
   5. Short CAT5e/CAT6 Patch Cables: To connect TVs, mesh nodes, and extra devices to the switches.

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
   Note: Because these devices pass directly through the switch to the fiber router, they will successfully live on the original network.

## Step 3: Connect the Secondary Mesh Router

   1. Take the Main Mesh Router node.
   2. Plug an ethernet cable from its WAN / Internet port into Port 7 of the 8-port switch.
   3. Power on the Mesh Router and follow its mobile app setup to create your Secondary Wi-Fi SSID.

## Step 4: Wire the Mesh Satellites (Hardwired Backhaul)

   1. Locate the LAN ports on the back of your Main Mesh Router.
   2. Run an ethernet cable from Mesh Router LAN Port 1 directly to Mesh Satellite 1.
   3. Run an ethernet cable from Mesh Router LAN Port 2 directly to Mesh Satellite 2.
   Warning: Do not plug the satellites into the 8-port switch. They must plug directly into the Main Mesh Router's LAN ports so they can communicate on the secondary network properly.

------------------------------
## Crucial Configuration Tips

* Avoid IP Conflicts (Double NAT): Because you have two routers (the Fiber Router and the Mesh Router), your network might experience "Double NAT," which can interfere with online gaming or smart home devices. If you encounter issues, log into your Mesh system's app and switch its operating mode from Router Mode to AP (Access Point) Mode or Bridge Mode. This keeps your secondary SSID active while letting the fiber router safely handle all network traffic.
* Cable Selection: For the long run between the router and the switch, ensure you use at least CAT5e (supports up to 1 Gbps up to 328 feet) or CAT6 (better shielding against interference).



