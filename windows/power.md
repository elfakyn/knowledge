Applies to: Windows 10, probably also 7/8/8.1

# Sleep

Sleep is janky in windows. Certain settings will cause any of this to happen:

## Problems

- Classic Start doesn't show the "sleep" option
    - see "low power idle"
- The computer overheats during sleep
    - see "low power idle"
- The computer randomly wakes up from sleep
    - see "wake timers"
- The computer goes to sleep soon after you lock it
    - see "system unattended sleep timeout"
    
## Solutions

### Low power idle

#### Check

To check if you have this problem: From an admin powershell run `powercfg -a`. If sleep state `Standby (S0 Low Power Idle)` is available, then you have this problem.

#### Fix

To fix it, open regedit and set `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\CsEnabled` to `0`. Reboot.

Now `powercfg -a` should show one of the other standby states: S1, S2, or S3. If it doesn't, you're SOL.

#### Explanation

In this configuration, the computer executes a "connected standby" when it's asked to sleep. Processes still run but on certain laptops fans may not spin. Windows still draws an obscene amount of power. Other standby modes are disabled by Windows when this is enabled, which means that even if the system is capable of a more reasonable sleep, then it still won't do it.

### Wake timers

#### Check

If your windows machine randomly wakes up from sleep, you have this problem. It's either windows update, or something else.

From admin powershell run `powercfg -waketimers`. It should show what's the problem.

#### Fix

If you want to only ban windows update from doing this, check here (COMING SOON). If you want to ban everything from waking your computer up, read on.

Go to Control Panel -> Power Options -> Change plan settings -> Change advanced power settings -> Sleep -> Allow wake timers. Set to Disable.

If the option isn't available, open regedit and change `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\BD3B718A-0680-4D9D-8AB2-E1D2B4AC806D\Attributes` to `2`. Close and open the advanced power settings and you should see the option.

#### Explanation 

to be continued
