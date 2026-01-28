# eSim — Ubuntu 25.04 Compatibility Guide

The following document summarizes the known issues when installing **eSim** on Ubuntu 25.04, the root causes, fixes, and engineering insights. After applying these fixes, eSim runs successfully with NGHDL, KiCad, and all dependencies on Ubuntu 25.04.

---
## Installation Step

***To install eSim and other dependencies, run the following command :***
```
./install-eSim.sh --install
```

***Through Terminal***

``` 
esim
```
---
## 🟢 Overview

Ubuntu 25.04 introduced multiple **dependency and version changes**:

* Hard-coded Ubuntu version checks in scripts
* KiCad PPA unavailable
* GTK modules renamed/removed
* NGHDL and GHDL toolchain mismatches
* Configuration paths changed for KiCad

This guide documents **all issues, fixes, and engineering rationale**.

---
## **Assessment:**

* 6 issues reported & fixed
* **High difficulty:** Issue 2 (KiCad PPA), Issue 6 (GHDL + LLVM)
* **Medium difficulty:** Issue 1 (OS check), Issue 4 (NGHDL)
* **Low difficulty:** Issue 3 (KiCad config), Issue 5 (GTK Canberra)

---
## ⚠️ Issue 1 — Ubuntu Version Check Fails for 25.04

[Detailed Issue 1 Document](./Issues/Issue-1.md)

**Problem:** The installer script does not recognize Ubuntu 25.04 and exits immediately.

**Error:**

```text
Detected Ubuntu Version:
Unsupported Ubuntu version: 25.04 ()
```

**Fix:**
Edit `install-eSim.sh`:

```bash
# Old
"24.04")
      SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
      ;;

# New
"24.04"|"25.04")
      SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
      ;;
```

---

## ⚠️ Issue 2 — KiCad PPA Not Available

[Detailed Issue 2 Document](./Issues/Issue-2.md)

**Problem:** KiCad PPA (`kicad/kicad-6.0-releases`) does not exist on Ubuntu 25.04.

**Fix:** Install KiCad from the **official repository**:

```bash
elif [[ "$ubuntu_version" == "25.04" ]]; then
    echo "Ubuntu 25.04 detected. Installing KiCad from official repository (no PPA)."
    sudo add-apt-repository --remove -y ppa:kicad/kicad-6.0-releases 2>/dev/null
    sudo apt-get update
    sudo apt-get install -y kicad kicad-footprints kicad-libraries kicad-symbols kicad-templates
    echo "KiCad installation completed successfully!"
    return
else
    kicadppa="kicad/kicad-6.0-releases"
fi
```

**Command to remove old PPA:**

```bash
sudo add-apt-repository --remove ppa:kicad/kicad-6.0-releases
sudo apt clean
sudo apt update
```

---

## ⚠️ Issue 3 — KiCad Config Path Missing / Version Mismatch

[Detailed Issue 3 Document](./Issues/Issue-3.md)

**Problem:** Scripts assume `~/.config/kicad/6.0`, but KiCad 7/8 uses `~/.config/kicad/`.

**Fix:** Use version-independent path:

```bash
KICAD_CFG="$HOME/.config/kicad"

if [ -d "$KICAD_CFG" ]; then
    echo "kicad config folder exists: $KICAD_CFG"
else
    echo "Creating kicad config folder: $KICAD_CFG"
    mkdir -p "$KICAD_CFG"
fi

# Copy custom eSim symbols
cp kicadLibrary/template/sym-lib-table "$KICAD_CFG/"
echo "Symbol table copied"
```

---

## ⚠️ Issue 4 — NGHDL Unsupported Ubuntu Version

[Detailed Issue 4 Document](./Issues/Issue-4.md)

**Problem:** NGHDL installer does not recognize Ubuntu 25.04.

**Fix:** Edit `install-nghdl.sh`:

```bash
case $VERSION_ID in
    "22.04")
        SCRIPT="$SCRIPT_DIR/install-nghdl-22.04.sh"
        ;;
    "23.04")
        SCRIPT="$SCRIPT_DIR/install-nghdl-23.04.sh"
        ;;
    "24.04"|"25.04")   # Added 25.04 here
        SCRIPT="$SCRIPT_DIR/install-nghdl-24.04.sh"
        ;;
    *)
        echo "Unsupported Ubuntu version: $VERSION_ID ($FULL_VERSION)"
        exit 1
        ;;
esac
```

**Patch NGHDL installation dynamically:**

```bash
sed -i 's/"24.04"/"24.04"|"25.04"/' install-nghdl.sh
chmod +x install-nghdl.sh
./install-nghdl.sh --install
```

---

## ⚠️ Issue 5 — GTK Canberra Module Not Available

[Detailed Issue 5 Document](./Issues/Issue-5.md)

**Problem:** `libcanberra-gtk-module` is missing on Ubuntu 25.04.

**Fix:** Replace hard-fail installation with graceful fallback:

```bash
echo "Installing Gtk Canberra modules (if available)..."

set +e  # Disable exit-on-error
sudo apt install -y libcanberra-gtk-module libcanberra-gtk3-module
if [ $? -ne 0 ]; then
    echo "Warning: libcanberra-gtk-module not found. Skipping..."
fi
set -e  # Re-enable exit-on-error
```

---

## ⚠️ Issue 6 — GHDL + LLVM Version Mismatch

[Detailed Issue 6 Document](../EsimFix/Issues/Issue-6.md)

**Problem:** Ubuntu 25.04 ships LLVM 20.1.2, but GHDL 4.1.0 does not support LLVM 20.

**Fix:** Install LLVM 18 alongside LLVM 20 and force GHDL to use it.

```bash
# Add LLVM repo and install LLVM 18
sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"
sudo apt install -y llvm-18 llvm-18-dev clang-18 lldb-18 lld-18
```

**Prevent re-extracting NGHDL source:**

```bash
if [ ! -d "$HOME/$nghdl" ]; then
    tar -xJf $nghdl-source.tar.xz -C $HOME
    mv $HOME/$nghdl-source $HOME/$nghdl
else
    echo "NGHDL already extracted, skipping extraction"
fi
```

**Force LLVM 18 in configure:**

```bash
chmod +x configure
export LLVM_CONFIG=/usr/bin/llvm-config-18
export CC=clang-18
export CXX=clang++-18

./configure --with-llvm-config=/usr/bin/llvm-config-18 \
            --enable-xspice \
            --disable-debug \
            --prefix=$HOME/$nghdl/install_dir/ \
            --exec-prefix=$HOME/$nghdl/install_dir/
```

**Build & Install:**

```bash
make -j$(nproc)
sudo make install
./install-nghdl.sh --install
```

---

## 🏁 Conclusion

The successful adaptation of **eSim for Ubuntu 25.04** required a **multidisciplinary approach**, addressing challenges from OS detection to runtime configuration and build toolchains. Each fix demonstrates careful engineering and cross-domain problem-solving.

| **Skill Domain**                                     | **Relevant Issues** | **Engineering Competency Demonstrated**                                                                       |
| ---------------------------------------------------- | ------------------- | ------------------------------------------------------------------------------------------------------------- |
| **OS Compatibility / Scripting**                     | Issue 1, Issue 4    | Writing robust Bash scripts that handle new OS versions gracefully                                            |
| **Package Management / Dependency Resolution**       | Issue 2, Issue 5    | Managing package availability, safe fallbacks, avoiding broken dependencies                                   |
| **Application Configuration / Version Independence** | Issue 3, Issue 6    | Ensuring software works across multiple versions and adapting configuration paths dynamically                 |
| **Build Systems / Compiler Toolchains**              | Issue 6             | Compiling complex applications with explicit toolchain control and environment management                     |
| **Error Handling / Installer Reliability**           | Issues 1–6          | Implementing safe fallbacks, preventing overwrites, handling optional components, and improving user feedback |
| **Testing / Validation**                             | All Issues          | Verifying installation success, ensuring runtime usability, and maintaining backward compatibility            |

---

### ✅ Key Takeaways

* Minimal, targeted script modifications improve **cross-version support**.
* Explicit toolchain and configuration management prevents runtime failures.
* Installer robustness enhances user experience and reduces post-install work.
* Backward compatibility ensures **older and newer environments** remain supported.

---

### 📸 Screenshots

<div style="text-align: center;">
  <img src="./Issues/Images/Pasted image (7).png" style="margin: 10px; border: 2px solid red;"  />
  <img src="./Issues/Images/Pasted image (8).png" style="margin: 10px; border: 2px solid red;" />
  <img src="./Issues/Images/Pasted image (9).png" style="margin: 10px; border: 2px solid red;" />
</div>

---

### 🌟 Future Recommendations

* Automate Ubuntu version detection for future releases.
* Introduce dynamic LLVM version selection to reduce manual intervention.
* Centralize configuration management for KiCad/eSim settings.
* Maintain a version compatibility matrix for NGHDL, KiCad, and LLVM.

---

