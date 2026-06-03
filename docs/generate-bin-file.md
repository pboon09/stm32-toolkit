# Generate a `.bin` File in STM32CubeIDE (Ship Without Source)

STM32CubeIDE builds an `.elf` by default. A `.bin` or `.hex` file is just machine
code with no source, comments, or symbols, so you can hand it to someone to flash
without sharing your project.

## What you get

| File | What it is |
|------|-----------|
| `.elf` | The default build output used for debugging because it keeps symbols. |
| `.hex` | A shareable image that already contains its load address, so it is the easiest one to give away. |
| `.bin` | A shareable raw image that you must write to address `0x08000000`. |

## Steps

1. Right-click the project in **Project Explorer** and choose **Properties**.
2. Go to **C/C++ Build** > **Settings** > **Tool Settings** > **MCU Post build outputs**.
3. Check **Convert to binary file (-O binary)**. Also check **Convert to Intel Hex** if you want a `.hex`.
4. Click **Apply and Close**, then run **Clean Project** and **Build Project**.

The files appear next to the `.elf` inside the active build folder.

```
YourProject/Debug/YourProject.bin     # or Release/
YourProject/Debug/YourProject.hex
```

Build the **Release** configuration for distribution rather than Debug. You switch
it with **Project** > **Build Configurations** > **Set Active** > **Release**.

## Lock the firmware on-chip (optional)

This step stops other people from copying your program back off the board. A
`.bin` or `.hex` hides your source code, but the compiled program still sits inside
the chip flash, and anyone who plugs in a debugger can read it out. Turning on
Read-Out Protection blocks that read-back. Do this in STM32CubeProgrammer.

1. Plug the board into USB and open **STM32CubeProgrammer**.
2. In the top-right corner, select **ST-LINK** in the dropdown and click **Connect**.
3. On the left toolbar, click the **OB** icon, which opens the Option Bytes panel.
4. In the panel, expand the **Read Out Protection** section and find the row named **RDP**.
5. Open the **RDP** dropdown and select **Level 1**, which is shown as the value **BB**.
6. Click the **Apply** button at the top of the Option Bytes panel.
7. Click **Disconnect** when it finishes.

After this a debugger can no longer read the flash. If you ever need to remove the
protection you set RDP back to Level 0, but the chip erases all of its flash during
that change, so nobody can recover your firmware. Do not use Level 2, because it
locks the debug port forever and cannot be undone.

## If something fails

- If the **MCU Post build outputs** page is missing, you either opened the wrong Properties or the project is not an STM32 project, so right-click the project to open it.
- If no `.bin` appears after building, the incremental build skipped it, so run **Clean Project** and then **Build Project**.
- If the recipient's `.bin` will not run, remember that a `.bin` has no address, so give them the `.hex` or tell them to flash it to `0x08000000`.

## References

- [How to generate `.bin` after building with CubeIDE](https://community.st.com/t5/stm32cubeide-mcus/how-to-generate-bin-file-after-building-with-cubeide/m-p/334423)
- [Building `.bin` file in CubeIDE](https://community.st.com/t5/stm32cubeide-mcus/building-bin-file-cubeide/td-p/207139)
