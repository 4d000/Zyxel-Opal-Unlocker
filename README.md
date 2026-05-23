# Zyxel Opal Firmware Unlocker 🔓

This repository contains JavaScript snippets that bypass the GUI restrictions on ISP-provided Zyxel routers running the "Opal" firmware architecture (Vue.js frontend). 

These scripts were developed and tested against **Wind Italy (ABXA)** firmwares but should work on other locked Zyxel models.

### Tested Models & Firmware Versions
* **DX5401-B0** | FW `V5.17(ABXA.2)b9` (Strict hardware/software ISP binding)
* **DX3301-T0** | FW `V5.50(ABVY.7)C0` (EasyMesh architecture. *Note: This firmware is less restricted out-of-the-box as it lacks the strict WindTre customization.*)
* **VMG8825** | FW `V5.13(ABLZ.1)b10_20200422` & `V5.13(ABLZ.1)b8_20190225` (Legacy ZyMesh, softer UI-level locks.)

⚠️ **Disclaimer:** These scripts modify the router's running configuration. While the UI changes are temporary (until page refresh), clicking "Apply" saves the settings to the router's NVRAM. Messing with Interface Groups or TR-069 can break your ISP's VoIP, IPTV, or remote support services. Use at your own risk.

---

## 🧠 How the Locks Work
ISPs lock these routers using two primary mechanisms in the Vue.js frontend:
1. **State Flags (`guiFlag`):** Variables like `WindItalyCustomization` or `hideadvance` are set to `true` by the ISP profile, which hides buttons, tabs, and sidebar menus.
2. **Navigation Guards:** Vue Router `beforeHooks` intercept requests to advanced pages and redirect you back to the home page if you aren't an authorized "supervisor" user.

These scripts manipulate the Vue instance (`__vue__`) directly from the browser's Developer Console (F12) to elevate privileges and strip the guards.

---

## 🚀 The "Pre-Login" Bypass Tactic
I have discovered that for models like the **DX5401**, running the unlock scripts **before** authenticating (at the login screen) is the most effective method. By injecting the override flags before the login handshake completes, you prevent the router's backend from ever applying the ISP-specific "hide" logic during the session initialization.

**How to execute:**
1. Navigate to the router's login page (`/login`).
2. Open the browser console (F12).
3. Paste the **Master Unlock** script.
4. Log in normally. The advanced menus should now remain visible.

---

## 🚀 Core Scripts (Console Usage)

### 1. The Master Unlock (Kill Guards & Elevate Privilege)
Run this to kill routing security, elevate your user level to "supervisor", and strip ISP customization flags.

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
Once unlocked, force navigation via console:

```javascript
// Advanced Networking
document.querySelector('#app').__vue__.$router.push('/InterfaceGrouping');
document.querySelector('#app').__vue__.$router.push('/VlanGroup');

// ISP Remote Management
document.querySelector('#app').__vue__.$router.push('/TR069Client');
```

---

## 🕵️ Discovering Hidden Paths
You can map every available route on your specific firmware by running this in the console:

```javascript
(function() {
    const routes = document.querySelector('#app').__vue__.$router.options.routes;
    function printRoutes(r, indent = "") {
        r.forEach(route => {
            console.log(`${indent}${route.path}`);
            if (route.children) printRoutes(route.children, indent + "  ");
        });
    }
    printRoutes(routes);
})();
```

---

## 🛜 EasyMesh Menu & Role Unlocker
Forces the EasyMesh/MPro Mesh tab to appear and allows role selection (Controller/Agent).

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
                    alert("Mesh tab found and activated!");
                }
            }
        }, 500);
    });
})();
```

---

## ⚠️ Known Limitations
* **DX5401 Interface Grouping:** Hard-locked WAN creation. Edit existing profiles instead of creating new ones.
* **VMG8825:** Older ZyMesh architecture is less responsive to UI-level unlocks compared to EasyMesh.

---

## 🔖 Bookmarklets (Minified)
Use these for one-click access. Create a new bookmark, name it, and paste the code into the URL field.

**Master Unlock (Minified):**
`javascript:(function(){const root=document.querySelector('#app').__vue__;const flags=root.$store.state.guiFlag;root.$router.beforeHooks=[];root.$store.state.userLevel="supervisor";root.$store.state.curloginLevel=2;flags.WindItalyCustomization=false;flags.hideEthWanTab=false;flags.hideadvance=false;flags.apas=true;flags.apas_intfgrp=true;flags.ZYXEL_VLAN_GROUP=true;alert("Master Unlock complete!");})();`

**EasyMesh Unlocker (Minified):**
`javascript:(function(){const state=document.querySelector('#app').__vue__.$store.state;const flags=state.guiFlag;flags.showMeshLabel=true;flags.showEasyMeshLabel=true;flags.showController=true;flags.WindItalyCustomization=false;state.userLevel="supervisor";const root=document.querySelector('#app').__vue__;root.$children.forEach(child=>{if(child.$options.name==='MenuList'||child.$options.name==='Navbar'){child.$forceUpdate();}});root.$router.push('/Wireless').then(()=>{setTimeout(()=>{const view=root.$route.matched[0].instances.default;if(view&&view.tabContent){const meshTabIndex=view.tabContent.findIndex(t=>t.name.toLowerCase().includes('mesh'));if(meshTabIndex!==-1){view.tabActiveIndex=meshTabIndex;alert("Mesh tab activated!");}}},500);});})();`

---

## 📄 License
This project is licensed under the MIT License. See the LICENSE file for details.
