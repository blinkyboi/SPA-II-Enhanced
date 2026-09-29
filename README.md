# SPA-II Enhanced Compatibility Fix

This repository contains a compatibility-focused build of **Single Player Apartment (SPA-II)** for **GTA V Enhanced**.

It addresses two major issues encountered when running the original SPA-II v2.0.5 release under GTA V Enhanced with ScriptHookVDotNet Enhanced:

- Severe FPS drops caused by repeated decorator registration every game Tick.
- Garage vehicle duplication caused by unavailable/invalid GTA decorators in the SHVDN v2 compatibility environment.

The garage fix replaces decorator-dependent vehicle return tracking with an in-memory vehicle-handle identity map for two-car, six-car, and ten-car garages.

---

## Installation

If you downloaded the latest original SPA-II version, **v2.0.5**, from:

[Single Player Apartment (SPA-II) on GTA5-Mods](https://www.gta5-mods.com/scripts/single-player-apartment-spg-net)

then installation is simple:

1. Close GTA V completely.
2. Open this repository's compiled output folder:

   ```text
   SPAII\bin\Release
   ```

3. Copy the provided:

   ```text
   SPAII.dll
   ```

4. Paste it into your GTA V Enhanced `scripts` folder:

   ```text
   Grand Theft Auto V Enhanced\scripts\
   ```

5. Replace the existing `SPAII.dll` when prompted.
6. Start GTA V Story Mode.

No other SPA-II files need to be replaced when upgrading from the original v2.0.5 release.

> Always back up your existing `scripts\SPA II\Garages\` folder before testing a modified SPA-II build.

---

## Requirements

This project was tested with:

```text
GTA V Enhanced
Script Hook V
ScriptHookVDotNet Enhanced
ScriptHookVDotNet2.dll
ScriptHookVDotNet3.dll
SPA-II v2.0.5 base installation
```

SPA-II is a ScriptHookVDotNet v2 mod. GTA V Enhanced can run it through ScriptHookVDotNet Enhanced's v2 compatibility layer.

---

## Changes

### FPS fix: disable per-Tick decorator registration

The original SPA-II source attempts to register vehicle decorators every game Tick.

That creates severe frame drops under GTA V Enhanced. The repeated Tick calls have been commented out.

```vb
Private Sub SPA2_Tick(sender As Object, e As EventArgs) Handles Me.Tick
    ' Decorator registration is disabled for GTA V Enhanced / SHVDN v2 compatibility.
    ' Do NOT call RegisterDecor every Tick: it causes severe frame drops.
    ' Garage return identity uses non-decorator tracking instead.
    'RegisterDecor(vehIdDecor, Decor.eDecorType.Int)
    'RegisterDecor(vehUidDecor, Decor.eDecorType.Int)

    PP = Game.Player.Character
    LV = Game.Player.Character.LastVehicle
    PM = Game.Player.Money
    Player = Game.Player
    PI = Game.Player.Character.Position.GetInterior

    ' ... rest of code is unmodified ...
End Sub
```

### Why decorators are disabled

SPA-II originally relies on GTA decorators to identify stored garage vehicles:

```text
inmspa2id
inmspa2uid
```

Under the tested GTA V Enhanced / ScriptHookVDotNet v2 compatibility environment, those decorators do not register successfully and return `0` when read.

This caused SPA-II to see a returned garage car as a brand-new outside vehicle:

```text
FromApartment = 0
UniqueID = 0
```

The original XML record remained in the garage folder while SPA-II saved another XML record for the returning car, resulting in duplicated vehicles and full garage slots.

---

## Garage duplication fix

The modified build no longer depends on GTA decorators for vehicle identity during garage exits and returns.

Instead, it uses an in-memory identity map:

```text
Exterior vehicle handle
→ original ApartmentID
→ original saved vehicle UID
```

### Updated garage flow

```text
1. Player enters a saved SPA-II garage vehicle.
2. SPA-II identifies the exact interior vehicle by its GTA vehicle handle.
3. SPA-II resolves the matching saved garage slot and XML vehicle record.
4. SPA-II creates the exterior vehicle clone.
5. The clone's handle is mapped to its original ApartmentID and UniqueID.
6. When that exact exterior vehicle returns to a garage:
   - SPA-II retrieves its identity from the handle map.
   - SPA-II deletes the prior XML record.
   - SPA-II removes the exterior vehicle from the world tracking list.
   - SPA-II saves one updated vehicle record.
   - SPA-II loads the correct new interior vehicle by saved garage slot.
7. The temporary handle identity entry is removed.
```

### Supported garage types

The decorator-free garage identity flow is implemented for:

```text
Two-car garages
Six-car garages
Ten-car garages
```

### Result

This prevents duplicate vehicle records when returning a stored garage vehicle:

```text
Before:
Garage car → drive out → drive back in
→ original XML remains
→ new XML is created
→ duplicate car / duplicate garage slot

After:
Garage car → drive out → drive back in
→ original XML is removed
→ one replacement XML is saved
→ one car remains in the garage
```

---

## Restart behavior

The vehicle-handle identity map is intentionally session-only.

If a saved SPA-II car is driven out and GTA V is restarted before it is returned:

```text
The original garage XML still exists.
The exterior mod-created vehicle is removed on restart.
The car reloads in its original garage.
```

If the car is returned during the same GTA V session:

```text
The original XML is replaced.
The returned car is stored once.
No duplicate is created.
```

This behavior is intentionally conservative and protects saved garage vehicles from being lost.

---

## Testing performed

The following workflow was tested successfully with a ten-car garage:

```text
1. Enter a ten-car SPA-II garage.
2. Drive out a previously stored garage vehicle.
3. Restart GTA V.
4. Confirm the vehicle reloads in the garage.
5. Drive the same vehicle out again.
6. Return it to the same garage.
7. Confirm no duplicate vehicle appears.
8. Confirm no extra garage XML record is created.
```

Expected result:

```text
Same number of saved XML files
Same number of stored vehicles
No duplicate vehicle
No lost garage slot
```

---

## Notes

- Do not re-enable the per-Tick `RegisterDecor(...)` calls unless you are specifically testing decorator behavior.
- Re-enabling those calls may cause major FPS drops.
- GTA decorators are not used for garage return identity in this build.
- Always exit GTA V normally after important garage changes.
- Back up your garage data before installing experimental builds:

  ```text
  Grand Theft Auto V Enhanced\scripts\SPA II\Garages\
  ```

- Use the in-game SPA-II Vehicle Management → `Remove` option to remove any old duplicate vehicles created before installing this fix.

---

## Disclaimer

This project is a compatibility modification of the original SPA-II v2.0.5 release for GTA V Enhanced.

It is not affiliated with Rockstar Games, ScriptHookV, ScriptHookVDotNet, or the original SPA-II author.

Use mods in GTA V Story Mode only. Do not use this or any other ScriptHook-based modification in GTA Online.
