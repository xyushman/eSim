# ⚠️ Issue 1 — Ubuntu Version Check Fails for 25.04

![Installer blocked on Ubuntu 25.04](./Images/Pasted%20image.png)

---

## ❌ Observed Behavior

The installer immediately terminates and reports Ubuntu 25.04 as unsupported.

### Terminal Output

```text
Detected Ubuntu Version:
Unsupported Ubuntu version: 25.04 ()
```

As a result, no dependencies are installed and the installation process does not proceed further.

---


## 📍 Faulty Code Location

**File:** `install-eSim.sh`
**Section:** Ubuntu version detection logic

### Original Code (Before Fix)

```bash
"24.04")
      SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
      ;;
```

Ubuntu 25.04 is not handled, causing the installer to abort.

---

## 🛠️ Fix Applied

Ubuntu 25.04 is mapped to the existing Ubuntu 24.04 installation routine, which is fully compatible.

### Modified Code (After Fix)

```bash
"24.04"|"25.04")
      SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
      ;;
```

This enables forward compatibility without duplicating installer scripts.

---

## 🧪 Validation & Testing

After applying the fix:

1. Re-ran the installer:

   ```bash
   ./install-eSim.sh --install
   ```
2. Ubuntu 25.04 was correctly detected
3. Installer proceeded to dependency installation
4. No regression observed on Ubuntu 24.04

---



