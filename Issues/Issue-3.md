Here’s your Issue 3 documentation cleaned up, structured, and fully professional, keeping all the technical details intact:

---

# Issue 3 — KiCad Config Path Missing / Version Mismatch

## Observed Behavior

The installer assumes the KiCad configuration path is:

```
~/.config/kicad/6.0
```

This fails on newer KiCad versions.
![imag](./Images/Pasted%20image%20\(3\).png)

**Symptom:**
The installer prints:

```
.config/kicad/6.0 does not exist
symbol table copied in the directory
```

even though KiCad appears to be installed.

---

## Cause

* The script assumes KiCad 6 path: `~/.config/kicad/6.0`.
* Newer KiCad versions (7/8) use a version-independent folder: `~/.config/kicad/`.

---

## Impact

* The `sym-lib-table` may not load correctly in the KiCad GUI.
* Custom eSim symbols may not appear automatically.

**Severity:** Minor — installation succeeds, but usability is affected if not fixed.

---

## Fix

Use a version-independent configuration folder:

```bash
KICAD_CFG="$HOME/.config/kicad"
mkdir -p "$KICAD_CFG"
cp kicadLibrary/template/sym-lib-table "$KICAD_CFG/"
```

---

## Simple & Correct Fix (Minimal Changes)

### Steps

Replace **only** the following part of the installer script:

**Old code:**

```bash
if [ -d ~/.config/kicad/6.0 ];then
    echo "kicad config folder already exists"
else 
    echo ".config/kicad/6.0 does not exist"
    mkdir -p ~/.config/kicad/6.0
fi

cp kicadLibrary/template/sym-lib-table ~/.config/kicad/6.0/
```

**New code (version-independent):**

```bash
KICAD_CFG="$HOME/.config/kicad"

if [ -d "$KICAD_CFG" ]; then
    echo "kicad config folder exists: $KICAD_CFG"
else
    echo "Creating kicad config folder: $KICAD_CFG"
    mkdir -p "$KICAD_CFG"
fi

# Copy symbol table for eSim custom symbols
cp kicadLibrary/template/sym-lib-table "$KICAD_CFG/"
echo "symbol table copied in the directory"
```

---