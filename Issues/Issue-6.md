# Issue 6 — GHDL + LLVM Version Mismatch

## Observed Behavior

When installing GHDL 4.1.0 on Ubuntu 25.04, the installer fails with an error such as:

```
Unhandled version llvm 20.1.2
```

![img](./Images/Pasted%20image%20\(6\).png)

The installation aborts with:

```
Error! Kindly resolve above error(s) and try again.
Aborting Installation...
```

---

## Cause

* Ubuntu 25.04 ships with **LLVM 20.1.2**.
* GHDL 4.1.0 does **not yet support LLVM 20.x**.
* The `./configure` script in GHDL only recognizes older LLVM versions (12–15 depending on patch level).
* LLVM’s API can change between versions, so older GHDL versions cannot compile against newer LLVM releases.

---

## Impact

* GHDL fails to compile.
* NGHDL/eSim simulations cannot run until the LLVM mismatch is resolved.

**Severity:** High — the simulator cannot function without resolving the version mismatch.

---

## Solutions

You have three options:

### Option 1 — Install an Older Supported LLVM (Stable)

1. Install a compatible LLVM version, e.g., LLVM 14 or 15:

```bash
sudo apt install llvm-15 llvm-15-dev clang-15
```

2. Configure GHDL to use the installed LLVM:

```bash
./configure --with-llvm-config=/usr/bin/llvm-config-15
make -j$(nproc)
sudo make install
```

**Pros:** Stable, fully supported by GHDL 4.1.0.
**Cons:** You must manage multiple LLVM versions if other programs require newer LLVM.

---

### Option 2 — Install LLVM 18 Alongside LLVM 20 (Recommended)

This approach allows the system LLVM 20 to remain intact for other applications while GHDL uses LLVM 18.

**Step 1: Install LLVM 18**

```bash
# Add the official LLVM repository
sudo bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"

# Install LLVM 18 and related tools
sudo apt install -y llvm-18 llvm-18-dev clang-18 lldb-18 lld-18

# Verify installation
llvm-config-18 --version
```

**Step 2: Configure GHDL to use LLVM 18**

Locate the `installGHDL` function in `install-nghdl.sh` and modify the configure line:

**Original:**

```bash
./configure --with-llvm-config=/usr/bin/llvm-config
```

**Updated for LLVM 18:**

```bash
./configure --with-llvm-config=/usr/bin/llvm-config-18
```

Then compile and install:

```bash
chmod +x configure
./configure --with-llvm-config=/usr/bin/llvm-config-18
make -j$(nproc)
sudo make install
```

**Pros:**

* System LLVM 20 remains intact.
* Minimal manual interference and reduced chance of breaking other software.

---

### Option 3 — Patch GHDL Source (Advanced)

* Modify `configure` or `configure.ac` to recognize LLVM 20.x.
* Not recommended unless you are comfortable with source code compilation and patching build scripts.

---

### Recommendation

**Option 2** (install LLVM 18 alongside LLVM 20) is the cleanest and safest solution for Ubuntu 25.04.

After applying this fix, GHDL 4.1.0 compiles successfully, and NGHDL/eSim simulations will work without errors.

---
