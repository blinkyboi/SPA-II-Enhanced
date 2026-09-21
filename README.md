If you downloaded the latest version (v2.0.5) from https://www.gta5-mods.com/scripts/single-player-apartment-spg-net then all you need to do is replace `SPAII.dll` currently in your 'scripts' folder with the one provided in 'SPAII\bin\Release'. Then reload the game. 
---
### Work done
Commented out two functions that call `RegisterDecor` function. See snippet below:
```c++
Private Sub SPA2_Tick(sender As Object, e As EventArgs) Handles Me.Tick
    ' Registered once in SPA2.New().
    ' Do NOT call RegisterDecor every Tick: causes severe frame drops.
    'RegisterDecor(vehIdDecor, Decor.eDecorType.Int)
    'RegisterDecor(vehUidDecor, Decor.eDecorType.Int)

    PP = Game.Player.Character
    LV = Game.Player.Character.LastVehicle
    PM = Game.Player.Money
    Player = Game.Player
    PI = Game.Player.Character.Position.GetInterior
    ... rest of code is unmodified ...
```
