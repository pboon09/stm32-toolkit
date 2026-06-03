# Generate a `.bin` File in STM32CubeIDE (Ship Firmware Without Source)

STM32CubeIDE builds an `.elf` by default. The `.elf` is fine for debugging, but
it carries debug symbols and is awkward to hand to someone who just needs to
flash a board. This guide makes the IDE also emit a plain `.bin` (and optionally
`.hex`) on every build, so you can distribute the **compiled firmware only** —
the recipient flashes it and runs your program without ever seeing the source.

> A `.bin`/`.hex` is machine code. It contains no C source, comments, or file
> names. If you also want to stop someone reading the firmware back **off the
> chip**, see [Step 4 — Lock the firmware on-chip](#step-4--lock-the-firmware-on-chip-optional).

---

## What you get

| File | Contains addresses? | Typical use |
|------|---------------------|-------------|
| `.elf` | yes | Debugging in the IDE (has symbols) |
| `.hex` | yes (Intel HEX) | Share + flash; load address is baked in |
| `.bin` | no (raw image) | Share + flash; you specify the load address (`0x08000000` for internal flash) |

For handing firmware to other people, `.hex` is the most foolproof because the
flashing tool reads the target address from the file. `.bin` is smaller but the
person flashing must know to write it to `0x08000000`.

---

## Step 1 — Open the right project properties

The post-build options only appear when you open properties from the project,
not from `File`.

1. In **Project Explorer**, right-click the project (the one with the `.ioc`).
2. Choose **Properties**.

> If you open `File` > `Properties` instead, the **MCU Post build outputs** page
> will be missing.

## Step 2 — Enable the binary output

Navigate to the post-build conversion options and tick the formats you want.

1. Go to **C/C++ Build** > **Settings** > **Tool Settings** tab.
2. Select **MCU Post build outputs**.
3. Check **Convert to binary file (-O binary)**. Optionally also check
   **Convert to Intel Hex file (-O ihex)**.
4. Click **Apply and Close**.

## Step 3 — Rebuild and confirm

The conversion runs as a post-build step, so a rebuild produces the files.

```bash
# In the IDE: right-click project > Clean Project, then Build Project
```

In the **Console** view you should see `objcopy` run after the link:

```
arm-none-eabi-objcopy  -O binary  YourProject.elf  "YourProject.bin"
arm-none-eabi-objcopy  -O ihex    YourProject.elf  "YourProject.hex"
...
Finished building: YourProject.bin
Finished building: YourProject.hex
```

The files land next to the `.elf`, inside the active build-configuration folder:

```
YourProject/Debug/YourProject.bin       # or Release/ if you built Release
YourProject/Debug/YourProject.hex
```

> **Build Release, not Debug, for distribution.** Switch the active configuration
> with **Project** > **Build Configurations** > **Set Active** > **Release**.
> Release is optimized and omits debug info; the `.bin`/`.hex` are what you ship.

## Step 4 — Lock the firmware on-chip (optional)

A `.bin`/`.hex` hides your *source*. It does **not** stop someone with the board
and a debugger from reading the *compiled firmware* back out of flash. To block
read-back, enable **Read-Out Protection (RDP)** in the chip's option bytes.

1. Open **STM32CubeProgrammer** and connect to the target (ST-LINK).
2. Go to the **Option Bytes** (OB) panel.
3. Set **RDP** to **Level 1** (`BB`), then **Apply**.

What this does and its caveats:

- **Level 1** disables debug-port access to flash. To go back to Level 0 the chip
  performs a **full mass erase**, so your firmware can't be dumped — but it also
  can't be recovered.
- **Level 2** is permanent and disables the debug interface entirely. Only use it
  if you fully understand it — there is no way back.

> Hand the recipient the `.hex` (or `.bin` + the `0x08000000` address). They flash
> with STM32CubeProgrammer or by dragging the file onto the board's USB mass-storage
> drive. They never need, and never receive, the source project.

---

## If something fails

- **No "MCU Post build outputs" page** — you opened `File` > `Properties`, or the
  project isn't an STM32 (CubeIDE) project. Reopen via right-click on the project
  in Project Explorer (Step 1).
- **No `.bin` after building** — the build was incremental and nothing recompiled.
  Run **Clean Project**, then **Build** so the post-build step re-runs.
- **`.bin` is huge (hundreds of MB)** — your link address starts in flash
  (`0x08000000`) but you converted a region that includes high-address RAM. Make
  sure you convert the project `.elf`, not a memory dump; a normal STM32 image is
  tens to a few hundred KB.
- **Recipient flashes `.bin` but nothing runs** — `.bin` has no address; it must be
  written to `0x08000000` (internal flash base). Use the `.hex` instead, which
  carries the address.

---

## References

- [STM32CubeIDE forum — How to generate `.bin` after building](https://community.st.com/t5/stm32cubeide-mcus/how-to-generate-bin-file-after-building-with-cubeide/m-p/334423)
- [STM32CubeIDE forum — Building `.bin` file in CubeIDE](https://community.st.com/t5/stm32cubeide-mcus/building-bin-file-cubeide/td-p/207139)
