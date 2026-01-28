# Issue 2 — KiCad PPA Not Available for Ubuntu 25.04

## Issue Found

When running the installer, you may encounter the following error:

```
https://ppa.launchpadcontent.net/kicad/kicad-6.0-releases/ubuntu plucky Release
404 Not Found
Aborting Installation...
```

![imag](./Images/Pasted%20image%20\(2\).png)

### Root Cause

The installer tries to add the PPA:

```
ppa:kicad/kicad-6.0-releases
```

However, this PPA **does not provide packages for Ubuntu 25.04 (Plucky)**.

As a result:

* `apt update` fails
* The installer aborts

---

## Option - 1: Simple Fix

Ubuntu 25.04 already includes KiCad in its **official repositories**.

**Solution:**

* Skip the PPA
* Install KiCad directly from the official repo

---

##  Option - 2:  Minimal & Correct Fix for Ubuntu 25.04 (Recommended)

### Goal

For Ubuntu 25.04 (Plucky):

* Do **not** use any PPA
* Install KiCad from the official Ubuntu repository

---

### Steps to Modify the Installer

1. Locate this section in your script:

```bash
    else
        kicadppa="kicad/kicad-6.0-releases"
    fi
```

2. Replace it with the following:

```bash
    elif [[ "$ubuntu_version" == "25.04" ]]; then
        echo "Ubuntu 25.04 detected. Installing KiCad from official repository (no PPA)."

        # Remove any previous PPA if exists
        sudo add-apt-repository --remove -y ppa:kicad/kicad-6.0-releases 2>/dev/null

        # Update package lists
        sudo apt-get update

        # Install KiCad and related packages
        sudo apt-get install -y kicad kicad-footprints kicad-libraries kicad-symbols kicad-templates

        echo "KiCad installation completed successfully!"
        return
    else
        kicadppa="kicad/kicad-6.0-releases"
    fi
```
---