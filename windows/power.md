Applies to: Windows 10, probably also 7/8/8.1. Check the last edited date on this document.

This guide assumes you know your way around Windows and have already tried the usual suspects.

# Windows sleep problems

## Rant (feel free to skip)

Windows power management is a janky piece of shit that's made worse by inconsistent vendor implementation. Every fucking hardware manufacturer has its own ideas of what "sleep" and "low power" and "wake" mean resulting in a fucking patchwork of non-standards that all abuse the already crappy power management system in all sorts of horrible fucking ways (how about TURN OFF THE FANS BUT KEEP THE CPU RUNNING IN "SLEEP" MODE). Goddess forbid you have a fucking OEM license preinstalled, fuck knows what settings they set to ridiculous values and hid from you. The wake timers for Windows Update are absolutely bonkers. The system won't fucking go to sleep, or goes to sleep when it shouldn't, or doesn't sleep properly, or doesn't wake properly, what the fuck. Even fucking Linux has better power management today, when 10 years ago closing your laptop lid was a one way ticket to kernel panic land.

## Problems

Problem | Solution
--- | ---
Classic Start doesn't show the "sleep" option | Low power idle
The computer overheats during sleep | Low power idle
The computer randomly wakes up from sleep | Wake timers
The computer goes to sleep soon after you lock it | System unattended sleep timeout / Console lock display off timeout
    
## Solutions

### Low power idle

#### Check

To check if you have this problem: From an admin powershell run `powercfg -a`. If sleep state `Standby (S0 Low Power Idle)` is available, then you have this problem.

#### Fix

To fix it, open regedit and set `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\CsEnabled` to `0`. Reboot.

Now `powercfg -a` should show one of the other standby states: S1, S2, or S3. If it doesn't, you're SOL and will have to choose between no sleep and S0 sleep.

#### Explanation

In this configuration, the computer executes a "connected standby" when it's asked to sleep. Processes still run but on certain laptops fans may not spin. Windows still draws an obscene amount of power. Other standby modes are disabled by Windows when this is enabled, which means that even if the system is capable of a more reasonable sleep, then it still won't do it. Applications such as Classic Start specifically look for S1, S2, or S3 and won't display the sleep option if it isn't one of these.

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

### System unattended sleep timeout / Console lock display off timeout

#### Check

You have this problem if your computer goes to sleep when you issue a lock command and wait a few minutes, even with sleep timeout set to Never.

#### Fix

Go to Control Panel -> Power Options -> Change plan settings -> Change advanced power settings -> Sleep -> System unattended sleep timeout. Set to 0.

Note that this will also cause the system to not go back to sleep anymore after a wake timer triggers. If it doesn't work make sure to revert this one.

If the option isn't available, set the registry key `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\238C9FA8-0AAD-41ED-83F4-97BE242C8F20\7bc4a2f9-d8fc-4469-b07b-33eb785aaca0\Attributes` to `2`. Close and open the advanced power settings and you should see the option.

If that doesn't work (such as on a Zenbook Pro Duo), set `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Power\PowerSettings\7516b95f-f776-4464-8c53-06167f40cc99\8EC4B3A5-6868-48c2-BE75-4F3044BE88A7\Attributes` to `2` then in advanced power settings set Display -> Console lock display off timeout to 0. To get the screen to still blank out after a while and avoid burn-in, set a blank screensaver with a timeout of however many minutes you want. This seems a bit finicky so test that the screensaver triggers both while unlocked and while locked before relying on it.

#### Explanation

Windows has this hidden config where the system sleeps when it's unattended. The only way to reveal the option is to edit the above registry key. The primary purpose of this key is to set the go-back-to-sleep timeout for when wake timers wake the computer, but it has the side effect of affecting the sleep timeout on the lockscreen on some computers. If that doesn't work, it may be the case that a screen off triggers sleep, in which case we disable the "display off" timeout and replace it with a blank screensaver which doesn't turn off the display, but simply sets it to black.
