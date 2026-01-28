# Issue 4 — NGHDL Unsupported Ubuntu Version

## Observed Behavior

When running the NGHDL installer on Ubuntu 25.04, the script fails with:

```
Detected Ubuntu Version: 
Unsupported Ubuntu version: 25.04 ()
```

![imag](./Images/Pasted%20image%20\(4\).png)

---

## Cause

* The `install-nghdl-*.sh` scripts support only specific Ubuntu versions: 22.04, 23.04, 24.04.
* Ubuntu 25.04 is not recognized by the script.
* The script uses `lsb_release -rs` to detect the Ubuntu version and does not find 25.04 in its supported list.

---

## Impact

* NGHDL **may still work**, but the installer does **not automatically configure paths or dependencies** for Ubuntu 25.04.
* Manual setup may be required (install dependencies, configure environment variables, etc.).

**Severity:** Medium — tool functionality depends on manual dependency handling.

---

## Bottom Line

The installer stops **because of the version check**, not due to an actual error.

You have three options:

1. Modify the installer script to recognize Ubuntu 25.04.
2. Use a supported Ubuntu version (22.04, 23.04, 24.04).
3. Bypass the automated installer and configure NGHDL manually.

---

## Why It Happens

The `install-nghdl.sh` script contains functions `get_ubuntu_version()` and `run_version_script()` that select the appropriate inner script (e.g., `install-nghdl-22.04.sh`).

Since Ubuntu 25.04 is not listed, the script exits with an error.

---

## How to Fix

### Option 1 — Patch After Extraction (Recommended)

Patch the installer **immediately after unzipping**:

```bash
unzip -o nghdl.zip
cd nghdl/

# Patch install-nghdl.sh for Ubuntu 25.04
sed -i 's/"24.04"/"24.04"|"25.04"/' install-nghdl.sh

chmod +x install-nghdl.sh
trap "" ERR
./install-nghdl.sh --install
trap error_exit ERR
```

This ensures that every time the zip is extracted, Ubuntu 25.04 is automatically treated like 24.04.

---

### Option 2 — Patch the Zip Itself

1. Extract `nghdl.zip`.
2. Edit `install-nghdl.sh` inside it to add Ubuntu 25.04 support.
3. Re-compress the zip.

This prevents `unzip -o` from overwriting your changes.

---

### Option 3 — Manual Installation

1. Skip `installNghdl` entirely.
2. Manually run your patched `install-nghdl.sh` from the `nghdl/` folder.

This method is more manual but guarantees your patched version is used.

---

### Recommendation

**Option 1** is the cleanest and easiest to maintain, particularly if performing multiple installs or testing.

**Result:** After applying this patch, NGHDL installs successfully without aborting on Ubuntu 25.04.

