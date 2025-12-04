# Advanced EMS Medical System for MTA:SA

A complete EMS (Emergency Medical Services) system for MTA:SA Roleplay servers, focused on immersive medical RP and EMS gameplay.

This resource replaces the default death flow with a **knockout + EMS revive** system, a modern **HTML5 EMS panel**, and an **interactive treatment mini‑game** for reviving and healing players.

---

## Features

- **Knockout System**
  - Players are knocked out instead of instantly dying when their health drops below a configurable threshold (`hpFallen`).
  - Knocked players are frozen, controls disabled, and a countdown timer starts.
  - If not treated in time, they fully die and lose inventory & money (configurable via existing inventory integration).
    <img width="1540" height="952" alt="knocked" src="https://github.com/user-attachments/assets/8c2d2358-8fed-4855-af0b-46be3f17a1f9" />


- **Cause‑of‑Death Detection**
  - Automatically detects and stores probable cause of injury:
    - Bullet wound
    - Melee trauma
    - Explosion
    - Burnt
    - Drowning
    - Fall impact
  - Cause is saved in element data (`ems.causeOfDeath`) and displayed to EMS in the revive panel.

- **EMS Right‑Click Revive & Heal Panel**
  - EMS in the configured ACL group can **right‑click** a nearby player to open a modern HTML5 treatment panel.
  - Panel shows:
    - Patient account name
    - Current health
    - Knocked/Conscious state
    - Detected cause of injury
  - Includes an **interactive treatment flow**: sequence‑based task using letters A–G.
  - After successful treatment:
    - **Revive** knocked‑out patients (restores them to 40% HP and normal state).
    - **Heal** conscious but injured players (restores HP to 100).

- **Immersive Treatment Mini‑Game**
  - Treatment section displays a random sequence of letters (A–G).
  - EMS must click the letters in the correct order for several rounds.
  - Mistakes reset the sequence, success unlocks Revive/Heal buttons.
    <img width="822" height="562" alt="revive panel" src="https://github.com/user-attachments/assets/bd3c59b8-aa32-412b-91c9-fcdcf0d03edb" />


- **Death Menu & EMS Calls**
  - Knocked players get a death menu with:
    - Call EMS
    - New Life (with cooldown and confirmation)
  - Death menu EMS call integrates into the existing EMS panel & call queue.

- **EMS Panel (Existing Integration)**
  - Includes HTML EMS panel (`medicalpanel.html`) with:
    - List of active EMS calls
    - EMS‑only chat
  - EMS can pick calls, get blips/markers, and respond.
<img width="1040" height="634" alt="ems panel" src="https://github.com/user-attachments/assets/b2a3a755-1771-418d-8005-5f5707d5dc94" />



- **Notifications & UI**
  - All EMS/medical messages use a unified notification system via `add:notification`.
  - Knock/Revive/Heal messages, EMS test messages, and help calls are consistent and non‑spammy.

- **ACL‑Based Permissions**
  - All EMS‑only actions (panel, chat, revive/heal, right‑click panel) are controlled via a single ACL group configured in `config.lua`.

---

## Requirements

- **MTA:SA 1.5+**
- A working Roleplay/Gameplay setup that supports:
  - `inventory` resource (optional but recommended; used to clear inventory on death).
  - A notification system listening to `add:notification` events.
- ACL access to create and manage a group for EMS (e.g. `EmsDuty`).

---

## File Structure (inside this resource)

```text
[players]/medical/
  config.lua                -- EMS ACL configuration
  config/
    settingS.lua            -- Server-side HP/time config for knock system
    settingC.lua            -- Client-side time display config
  sGaba.lua                 -- Core server logic: knock, revive, EMS calls, cause-of-death
  cGaba.lua                 -- Core client logic: death UI, knock FX, revive panel integration
  sMedicalPanel.lua         -- EMS panel ACL checks
  cMedicalPanel.lua         -- EMS panel client logic (EMS calls + chat)
  sEMSChat.lua              -- EMS chat + /emstest command
  deathmenu.html            -- Knock/death menu
  medicalpanel.html         -- EMS panel (calls + chat)
  revivepanel.html          -- EMS treatment / revive & heal panel
  meta.xml                  -- Resource definition
  font/, images/, shaders/  -- UI assets and effects
  docs/README.md            -- This file
```

---

## Installation

1. **Copy the resource**

   Place the `[players]/medical` folder into your server resources directory:

   ```text
   mods/deathmatch/resources/[players]/medical
   ```

2. **Configure ACL group**

   Open `config.lua` inside this resource:

   ```lua
   -- EMS permission configuration for the medical resource
   EMS_ACL_GROUP = "EmsDuty"  -- Change this to your EMS ACL group name
   ```

   Make sure that ACL.xml in your server has a group with that name and EMS accounts are added to it, for example:

   ```xml
   <group name="EmsDuty">
       <acl name="Default" />
       <object name="user.EMS_Doctor_Account" />
       <!-- Add more EMS accounts here -->
   </group>
   ```

3. **Ensure dependencies exist**

   - `inventory` resource (optional): used to clear inventory on death.
   - `add:notification` handler: your UI framework must listen for:
     ```lua
     addEvent("add:notification", true)
     -- arguments: player, message, type ("success"|"info"|"warn"|"error"), [persistent]
     ```

4. **Add resource to ACL if needed**

   If your server restricts access to certain functions, make sure the medical resource has appropriate rights.

5. **Start the resource**

   In `mtaserver.conf`:

   ```xml
   <resource src="[players]/medical" startup="1" protected="0" />
   ```

   Or start it in console:

   ```text
   start [players]/medical
   ```

---

## How It Works In‑Game

### Knockout

- When a player’s health drops below `hpFallen` (from `config/settingS.lua`):
  - They become **knocked** instead of dying.
  - `playerFallen` element data is set.
  - A countdown (dead time) starts; if it ends without treatment, the player dies.
  - The client shows:
    - Visual shader effect.
    - Death/knock menu.

### Cause-of-Death Tracking

- Each time a player takes damage, the server records the last damage source and environment.
- When the player is set fallen, the server classifies the likely cause:
  - Uses weapon ID and environment (water, fire, fall).
- This string is stored in `ems.causeOfDeath` and sent to the EMS revive panel.

### EMS Right‑Click Treatment Flow

1. **Player is knocked** (`playerFallen = true`).
2. EMS (with ACL permission) moves within 4 units of the patient.
3. EMS **right‑clicks** the patient.
4. Server validates:
   - EMS is in `EMS_ACL_GROUP`.
   - Target is a player and in range.
5. Client opens `revivepanel.html` for the EMS, showing:
   - Patient account name.
   - HP and state (Knocked/Conscious).
   - Cause of injury.
6. EMS completes the treatment sequence.
7. EMS chooses:
   - **Revive** (for knocked patients): player is restored to 40 HP and all knock effects are removed.
   - **Heal** (for conscious but injured players): player HP set to 100.

---

## Commands

### Player / EMS

- **`/192`**
  - Player uses this to call EMS.
  - All EMS in the ACL group receive a notification and a blip is created at the caller’s location.

- **Death menu buttons** (client‑side)
  - **Call EMS**: invokes the EMS call queue, visible in the EMS panel.
  - **New Life**: kills the player after a cooldown and cleans inventory/money via the inventory integration.


## Configuration

### EMS ACL Group

- `config.lua`:
  ```lua
  EMS_ACL_GROUP = "EmsDuty"
  ```
  Change this to match your server’s EMS group.

### Knockout Threshold & Timers

- `config/settingS.lua` (server):
  ```lua
  hpFallen = 15      -- HP at which the player is considered knocked
  deadTime = 900000  -- Time before auto-death after knock (ms)
  ```

- `config/settingC.lua` (client):
  ```lua
  deadTime = 900000  -- Must match server deadTime for timer display
  ```

Adjust these values to match your roleplay standards (hardcore vs casual).

---
