# Issue 5 — GTK Canbera Module Not Available

## Observed Behavior

During installation, you may see the following error:

```
Package libcanberra-gtk-module is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source
```

![img](./Images/Pasted%20image%20\(5\).png)

---

## Cause

* The installer attempts to install `libcanberra-gtk-module`, which is required by GTK applications to handle sounds and notifications.
* On Ubuntu 25.04, this package may no longer exist, has been renamed, or has been split into multiple packages.
* As a result, `apt` cannot find the package, causing the installer to abort.

---

## Impact

* Most GTK applications, including NGHDL and KiCad, will still function.
* Some sound notifications may be missing if the module is not installed.

**Severity:** Low to Medium — affects sound/notification functionality but does not break the software.

---

## Fix Options

### Option 1 — Install Replacement Packages

On newer Ubuntu versions, the package is split into:

```bash
sudo apt install libcanberra-gtk3-module libcanberra-gtk-module
```

Even if `libcanberra-gtk-module` is missing, installing `libcanberra-gtk3-module` usually satisfies GTK3 apps.

---

### Option 2 — Skip if Not Critical

* The module is primarily used for sound events.
* NGHDL and KiCad will mostly work without it.

---

### Option 3 — Enable the Universe Repository

Some packages may only exist in the `universe` repository:

```bash
sudo add-apt-repository universe
sudo apt update
sudo apt install libcanberra-gtk-module libcanberra-gtk3-module
```

After enabling `universe`, the installer should find the packages.

---

## Recommended Fix


### Installer Script Adjustment

In `./eSim-2.5/nghdl/install-nghdl-scripts`, replace the old code:

```bash
# Specific dependency for canberra-gtk modules
echo "Installing Gtk Canberra modules..........................."
sudo apt install -y libcanberra-gtk-module libcanberra-gtk3-module
```

with the new version that **handles missing packages gracefully**:

```bash
echo "Installing Gtk Canberra modules (if available)..."
set +e  # temporarily disable exit on error
sudo apt install -y libcanberra-gtk-module libcanberra-gtk3-module
if [ $? -ne 0 ]; then
    echo "Warning: libcanberra-gtk-module not found. Skipping..."
fi
set -e  # re-enable exit on error
```

*This ensures the installer does not abort if the package is missing.*

---

This makes Issue 5 fully documented, clear, and actionable.
