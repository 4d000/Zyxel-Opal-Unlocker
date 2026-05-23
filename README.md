# Zyxel Opal Firmware Unlocker 🔓

This repository contains JavaScript snippets that bypass the GUI restrictions on ISP-provided Zyxel routers running the "Opal" firmware architecture (Vue.js frontend). 

These scripts were developed and tested against **Wind Italy (ABXA)** firmwares but should work on other locked Zyxel models.

### Tested Models & Firmware Versions
* **DX5401-B0** | FW `V5.17(ABXA.2)b9` (Strict hardware/software ISP binding)
* **DX3301-T0** | FW `V5.50(ABVY.7)C0` (EasyMesh architecture. *Note: This firmware is less restricted out-of-the-box as it lacks the strict WindTre customization.*)
* **VMG8825** | FW `V5.13(ABLZ.1)b10_20200422` & `V5.13(ABLZ.1)b8_20190225` (Legacy ZyMesh, softer UI-level locks. *Note: Mesh unlock scripts are incompatible with these specific builds.*)

⚠️ **Disclaimer:** These scripts modify the router's running configuration. While the UI changes are temporary (until page refresh), clicking "Apply" saves the settings to the router's NVRAM. Messing with Interface Groups or TR-069 can break your ISP's VoIP, IPTV, or remote support services. Use at your own risk.

---

## 🧠 How the Locks Work
ISPs lock these routers using two primary mechanisms in the Vue.js frontend:
1. **State Flags (`guiFlag`):** Variables like `WindItalyCustomization` or `hideadvance` are set to `true` by the ISP profile, which hides buttons, tabs, and sidebar menus.
2. **Navigation Guards:** Vue Router `beforeHooks` intercept requests to advanced pages and redirect you back to the home page if you aren't an authorized "supervisor" user.

These scripts manipulate the Vue instance (`__vue__`) directly from the browser's Developer Console (F12) to elevate privileges and strip the guards.

---

## 🚀 Core Scripts (Console Usage)
*Use these by opening your browser's Developer Tools (F12) and pasting them into the Console.*

### 1. The Master Unlock (Kill Guards & Elevate Privilege)
Run this from the home page. It kills the routing security, elevates your user level to "supervisor", and strips the ISP customization flags.

```javascript
javascript:(function() {
    const root = document.querySelector('#app').__vue__;
    const flags = root.$store.state.guiFlag;

    // 1. Kill Navigation Guards
    root.$router.beforeHooks = [];

    // 2. Elevate Privilege
    root.$store.state.userLevel = "supervisor";
    root.$store.state.curloginLevel = 2;

    // 3. Strip ISP Customizations
    flags.WindItalyCustomization = false;
    flags.hideEthWanTab = false;
    flags.hideadvance = false;
    flags.apas = true;
    flags.apas_intfgrp = true;
    flags.ZYXEL_VLAN_GROUP = true;

    alert("Master Unlock complete! You can now navigate to hidden URLs.");
})();
```

### 2. Direct Navigation to Hidden Pages
Once the Master Unlock is run, some menu items still won't appear in the sidebar. You can force the router to load the page directly by running one of these in the console:

```javascript
// Advanced Networking
document.querySelector('#app').__vue__.$router.push('/InterfaceGrouping');
document.querySelector('#app').__vue__.$router.push('/VlanGroup');

// ISP Remote Management (TR-069)
document.querySelector('#app').__vue__.$router.push('/TR069Client');
```

#### 💡 Why `/TR069Client` is interesting:
TR-069 (CWMP) is the protocol your ISP uses to remotely manage, update, and monitor your router. By accessing this hidden menu, you can see the ACS URL your ISP uses, view the connection status, or disable remote management entirely if you want to stop the ISP from pushing unwanted firmware updates or resetting your custom configuration.

---

## 🕵️ Discovering & Testing Hidden Paths
Because different firmware versions hide different features, you can query the router's internal Vue Router to print out a map of every single page that exists on the device, even the hidden ones!

**1. Run the Path Discovery Script:**

```javascript
(function() {
    const routes = document.querySelector('#app').__vue__.$router.options.routes;
    console.log("%c--- All Possible Router Paths ---", "color: orange; font-size: 14px; font-weight: bold;");
    
    function printRoutes(r, indent = "") {
        r.forEach(route => {
            console.log(`${indent}${route.path}`);
            if (route.children) printRoutes(route.children, indent + "  ");
        });
    }
    printRoutes(routes);
})();
```

**2. Test the Paths:**
Look through the list printed in your console. See something interesting like `/EoGRE` or `/Avast`? Test it using the direct navigation script:
```javascript
document.querySelector('#app').__vue__.$router.push('/YourDiscoveredPath');
```

---

## 🛜 EasyMesh Menu & Role Unlocker (DX3301 / Modern Firmwares)
ISPs often hardcode the router to act only as a Mesh Controller and hide the menu. This script forces the EasyMesh/MPro Mesh tab to appear, unlocks the global visibility flags, and allows you to set the router as an Agent (satellite).

```javascript
javascript:(function() {
    const state = document.querySelector('#app').__vue__.$store.state;
    const flags = state.guiFlag;

    flags.showMeshLabel = true;
    flags.showEasyMeshLabel = true;
    flags.showController = true;
    flags.WindItalyCustomization = false;
    state.userLevel = "supervisor";

    const root = document.querySelector('#app').__vue__;
    root.$children.forEach(child => {
        if (child.$options.name === 'MenuList' || child.$options.name === 'Navbar') {
            child.$forceUpdate();
        }
    });

    root.$router.push('/Wireless').then(() => {
        setTimeout(() => {
            const view = root.$route.matched[0].instances.default;
            if (view && view.tabContent) {
                const meshTabIndex = view.tabContent.findIndex(t => t.name.toLowerCase().includes('mesh'));
                if (meshTabIndex !== -1) {
                    view.tabActiveIndex = meshTabIndex;
                    alert("Mesh tab found and activated! You can now change roles.");
                }
            }
        }, 500);
    });
})();
```

---

## ⚠️ Known Limitations & Hardware Quirks

* **DX5401 Interface Grouping (FW V5.17(ABXA.2)b9):** If MPro Mesh is enabled, the firmware strictly locks the wireless interfaces (`wl0`, `wl1`) to the default bridge to maintain the backhaul. **You cannot group wireless networks into separate VLANs/Interfaces while MPro Mesh is enabled.** You must disable Mesh to isolate your WiFi networks.
* **VMG8825 Mesh Scripts:** The older ZyMesh architecture on the VMG8825 builds listed above responds poorly to UI-level unlocks for Mesh roles. The Mesh-specific scripts in this repo are geared toward newer EasyMesh/MPro Mesh architectures.

---

## 🔖 Bookmarklets (One-Click Unlocks)
You do not need to open the F12 Developer Console every time you want to use these. You can save them directly to your browser bookmarks!

*Note: Bookmark URLs cannot contain multi-line code or `//` comments, or they will throw an `Unexpected end of input` error. Use the **minified** scripts below for your bookmarks.*

**How to use:**
1. Right-click your browser's bookmark bar and click **Add Page** or **Add Bookmark**.
2. Name it appropriately (e.g., "Zyxel Master Unlock").
3. In the **URL** field, paste the minified code block below.
4. Click the bookmark while logged into your router.

**Master Unlock (Minified):**
```javascript
javascript:(function(){const root=document.querySelector('#app').__vue__;const flags=root.$store.state.guiFlag;root.$router.beforeHooks=[];root.$store.state.userLevel="supervisor";root.$store.state.curloginLevel=2;flags.WindItalyCustomization=false;flags.hideEthWanTab=false;flags.hideadvance=false;flags.apas=true;flags.apas_intfgrp=true;flags.ZYXEL_VLAN_GROUP=true;alert("Master Unlock complete! You can now navigate to hidden URLs.");})();
```

**EasyMesh Unlocker (Minified):**
```javascript
javascript:(function(){const state=document.querySelector('#app').__vue__.$store.state;const flags=state.guiFlag;flags.showMeshLabel=true;flags.showEasyMeshLabel=true;flags.showController=true;flags.WindItalyCustomization=false;state.userLevel="supervisor";const root=document.querySelector('#app').__vue__;root.$children.forEach(child=>{if(child.$options.name==='MenuList'||child.$options.name==='Navbar'){child.$forceUpdate();}});root.$router.push('/Wireless').then(()=>{setTimeout(()=>{const view=root.$route.matched[0].instances.default;if(view&&view.tabContent){const meshTabIndex=view.tabContent.findIndex(t=>t.name.toLowerCase().includes('mesh'));if(meshTabIndex!==-1){view.tabActiveIndex=meshTabIndex;alert("Mesh tab found and activated! You can now change roles.");}}},500);});})();
```

**Page Activator (Un-grey buttons - Minified):**
```javascript
javascript:(function(){const view=document.querySelector('#app').__vue__.$route.matched[0].instances.default;if(view){view.ControllDisabled=false;view.IsReadonly=false;if(view.guiConfig){view.guiConfig.showAdd=true;view.guiConfig.showEdit=true;view.guiConfig.showDelete=true;view.guiConfig.isReadonly=false;}view.$forceUpdate();alert("Page editing unlocked!");}})();
```

---

## 📄 License
This project is licensed under the MIT License. You are free to use, modify, and distribute these scripts as long as the original copyright notice is included. See the `LICENSE` file for details.
