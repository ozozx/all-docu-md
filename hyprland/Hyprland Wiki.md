---
subpages:
  - "[[hyprland/Getting Started/Getting Started]]"
  - "[[Configuring]]"
  - "[[hyprland/Useful Utilities/Hypr Ecosystem|Hypr Ecosystem]]"
  - "[[Useful Utilities]]"
  - "[[hyprland/Plugins/Plugins|Plugins]]"
  - "[[Nix]]"
  - "[[Nvidia]]"
  - "[[IPC]]"
  - "[[Crashes and Bugs]]"
  - "[[FAQ]]"
  - "[[Connect]]"
  - "[[hyprland/Contributing and Debugging/Contributing and Debugging|Contributing and Debugging]]"
---


Hello there, dear traveler! Welcome to the Hyprland Wiki!

Take a tour of the pages on the left and read ones that you may need.

***This wiki is versioned!*** By default, you are reading the wiki for _the latest Hyprland git commit_.
Click on the version selector and select your version if you are running a tagged release
(which you very likely are, you can check with `hyprctl version`).

## Wayland info (especially useful for Xorg users)

A Wayland compositor is a fully autonomous Display Server, like Xorg itself. It
is **not** possible to mix'n'match Wayland compositors like you could on Xorg
with window managers and compositors. It is also not entirely possible, nor
recommended, to try and use all Xorg applications on Wayland. See
[[Useful Utilities|this page]] for a list of recommended Wayland
native/compatible programs.

Wayland **compositors** should not be confused with Xorg **window managers**.

## IMPORTANT

If you are having issues, please try [[FAQ|reading the FAQ]] and configuring
sections - chances are your issue is described somewhere there. If not, you can
try [searching the issues](https://github.com/hyprwm/Hyprland/issues).
