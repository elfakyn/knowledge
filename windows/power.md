Applies to: Windows 10, probably also 7/8/8.1

This guide assumes you know your way around Windows and have already tried the usual suspects.

# Sleep

Sleep is janky in windows. Certain settings will cause any or all of this to happen:

## Problems

Problem | Solution
--- | ---
Classic Start doesn't show the "sleep" option | Low power idle
The computer overheats during sleep | Low power idle
The computer randomly wakes up from sleep | Wake timers
The computer goes to sleep soon after you lock it | System unattended sleep timeout
    
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

From admin shell run `powercfg -waketimers`. It should show what's the problem.

#### Fix

If you want to only ban windows update from doing this, check here (COMING SOON). If you want to ban everything from waking your computer up, read on.

Go to Control Panel -> Power Options -> Change plan settings -> Change advanced power settings -> Sleep -> Allow wake timers. Set to Disable.

If the option isn't available, set the registry key `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\BD3B718A-0680-4D9D-8AB2-E1D2B4AC806D\Attributes` to `2`. Close and open the advanced power settings and you should see the option.

#### Explanation

Windows mostly uses wake timers with Windows Update, which is usually the primary culprit, however other systems may rely on wake timers themselves. The above registry key controls whether the option is visible or not. You can find a list of other interesting registry keys by running `powercfg -q` (query). That list may not be exhaustive, [here's an exhaustive one](https://bitsum.com/known-windows-power-guids/).

### System unattended sleep timeout

#### Check

You have this problem if your computer goes to sleep when you issue a lock command and wait a few minutes.

#### Fix

Go to Control Panel -> Power Options -> Change plan settings -> Change advanced power settings -> Sleep -> System unattended sleep timeout. Set to 0.

If the option isn't available, set the registry key `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\7bc4a2f9-d8fc-4469-b07b-33eb785aaca0\Attributes` to `2`. Close and open the advanced power settings and you should see the option.

#### Explanation

Windows has this hidden config where the system sleeps when it's unattended. The only way to reveal the option is to edit the above registry key. It's the same for all computers. You can find a list of other interesting registry keys by running `powercfg -q` (query). That list may not be exhaustive, [here's an exhaustive one](https://bitsum.com/known-windows-power-guids/).
