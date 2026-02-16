![](https://i.ibb.co/dsvrXgb1/Gemini-Generated-Image-a8cyn9a8cyn9a8cy.png)

# Auto Power Profile Switcher for COSMIC DE on Fedora Atomic

This guide provides a robust solution to automatically switch power profiles (**Performance** / **Balanced**) on **COSMIC DE** running on **Fedora Atomic (Silverblue, Kinoite, Sericea)**, based on the power supply state (AC/Battery).

## Prerequisites

* **Fedora 43 Atomic** (or higher) with the COSMIC desktop environment installed.
* Administrative privileges (`sudo`).
* Terminal access.

## Step-by-Step

### 1. Power Backend Adjustment

Fedora 43 defaults to `tuned-ppd`. To ensure full compatibility with COSMIC DE and our `udev` automation rules, it is necessary to replace it with the original `power-profiles-daemon`.

```bash
# Perform the atomic driver substitution
rpm-ostree override remove tuned-ppd --install power-profiles-daemon

# Reboot to apply the new system layer
systemctl reboot

```

### 2. Creating the Udev Rule

`udev` rules detect hardware changes and execute commands in real-time.

1. Create the configuration file:
```bash
sudo nano /etc/udev/rules.d/99-powertuning.rules

```


2. Add the following instructions:
```text
# Switch to 'performance' when plugged in (AC)
SUBSYSTEM=="power_supply", ATTR{online}=="1", RUN+="/usr/bin/powerprofilesctl set performance"

# Switch to 'balanced' when unplugged (Battery)
SUBSYSTEM=="power_supply", ATTR{online}=="0", RUN+="/usr/bin/powerprofilesctl set balanced"

```


3. Reload the rules for immediate activation:
```bash
sudo udevadm control --reload-rules && sudo udevadm trigger

```

## Validation

To verify if the automation is working, monitor the profiles while plugging and unplugging your charger:

```bash
watch powerprofilesctl

```

The asterisk (`*`) should automatically toggle between profiles according to the power state.

---

## Credits and Tools

This tutorial utilizes and integrates the following third-party technologies:

* **[COSMIC DE](https://github.com/pop-os/cosmic):** A modern, modular desktop environment developed by **System76**, written in Rust.
* **[Fedora Atomic Desktops](https://fedoraproject.org/atomic-desktops/):** An immutable variant of Fedora that uses **rpm-ostree** for system-wide layered package management.
* **[power-profiles-daemon](https://gitlab.freedesktop.org/upower/power-profiles-daemon):** A **Freedesktop.org** project that provides a D-Bus interface for modifying hardware power characteristics.
* **[systemd/udev](https://systemd.io/):** The Linux device manager, essential for detecting hardware events.

---

## Implementation Notes

* **Why not use TLP?** COSMIC is designed to communicate natively with the `power-profiles-daemon`. Using TLP may lead to UI conflicts and erratic CPU scaling behavior.
* **Persistence:** On Fedora Atomic, the `/etc` directory is persistent, ensuring that `udev` rules survive system version upgrades.

---

Would you like me to help you write a **Contribution Guide (CONTRIBUTING.md)** or a **Code of Conduct** for this repository?
