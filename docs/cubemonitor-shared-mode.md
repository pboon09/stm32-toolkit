# Run STM32CubeMonitor and IDE Debug at the Same Time (Shared ST-LINK)

By default STM32CubeMonitor takes the ST-LINK for itself, so if you try to debug
in the IDE while CubeMonitor is running, one of them fails to connect. Shared mode
routes both tools through the ST-LINK server so they can use the probe at the same
time. You can step through code in STM32CubeIDE while CubeMonitor live-plots your
variables.

## Why it is needed

| Protocol | How it connects | Can it share the probe |
|----------|-----------------|------------------------|
| `p2p` (default) | It connects straight to the probe and gives the highest data rate. | No, because it locks out every other application. |
| `tcp` | It connects through the ST-LINK server, which shares access between apps. | Yes, and this is the mode that shared mode uses. |

Shared mode means you run CubeMonitor in `tcp` mode and tell the IDE to use the
shared ST-LINK. A J-Link probe supports `p2p` only, so shared mode is an ST-LINK
feature.

## Step 1 — Make sure the ST-LINK server is installed

Shared mode passes through a helper called the ST-LINK server, which normally
installs together with STM32CubeIDE and STM32CubeProgrammer. If you have either
tool you already have it. On Windows you can find it inside Program Files in a
folder named `stlink_server`.

## Step 2 — Switch CubeMonitor to `tcp` mode

The protocol is set in CubeMonitor's `settings.js` file, and the change only takes
effect the next time the app starts.

1. Close STM32CubeMonitor if it is open.
2. Open the `settings.js` file for your operating system from the table below.
3. Find the line that starts with `connectionType` and change its value to `tcp`.
4. Save the file and start STM32CubeMonitor again.

| Operating system | settings.js location |
|------------------|----------------------|
| Windows | `%USERPROFILE%\.STMicroelectronics\stm32cubemonitor\settings.js` (i.e. `C:\Users\<you>\.STMicroelectronics\stm32cubemonitor\settings.js`) |
| Linux | `~/.STMicroelectronics/stm32cubemonitor/settings.js` |
| macOS | `~/Library/stm32cubemonitor/settings.js` |

```js
connectionType: "tcp",
```

## Step 3 — Turn on shared mode in STM32CubeIDE

1. Open **Run** > **Debug Configurations**.
2. Select your debug configuration on the left.
3. Open the **Debugger** tab.
4. Check **Enable shared ST-LINK**.
5. Click **Apply**, then click **Debug** to start the session.

## Step 4 — Start the acquisition in CubeMonitor

With the debug session running, start your acquisition in CubeMonitor as usual by
pressing the start button on the acquisition node and opening the dashboard. When
it works the probe in the flow shows the text `tcp connected`, and both tools keep
running together.

## Other IDEs

| IDE | Where to switch it on |
|-----|-----------------------|
| STM32CubeIDE | Debug Configurations > Debugger tab > **Enable shared ST-LINK** |
| IAR Embedded Workbench | Project > Options > Debugger > Extra Options, then add `--stlink_use_server` |
| Keil uVision | Project > Options for Target > Debug tab > ST-Link settings, then set Debug adapter to **Shareable ST-Link** |

## If something fails

- If CubeMonitor still cannot connect while the IDE is debugging, the `connectionType` is probably still `p2p` or you did not restart CubeMonitor after the edit, so redo Step 2 and start the app again.
- If the IDE debug session will not start while CubeMonitor is open, **Enable shared ST-LINK** is not checked or the ST-LINK server is not installed, so check Step 1 and Step 3.
- If it connects but the data is slow or drops samples, that is the cost of `tcp` mode, so lower the sampling rate or switch back to `p2p` when you do not need the IDE attached.
- If you use a J-Link probe, shared mode is not available because J-Link supports `p2p` only, so use an ST-LINK to debug and monitor together.

## References

- [STM32CubeMonitor wiki, How to configure shared mode](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_configure_shared_mode)
- [STM32CubeMonitor wiki, How to change general settings](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_change_general_settings)
