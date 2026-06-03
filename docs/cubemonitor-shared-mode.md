# Run STM32CubeMonitor and IDE Debug at the Same Time (Shared ST-LINK Mode)

By default STM32CubeMonitor grabs the ST-LINK **exclusively**, so if you try to
start an IDE debug session while CubeMonitor is acquiring (or vice-versa) one of
them fails to connect — they're fighting over the same probe. This guide enables
**shared mode**, which routes both tools through the ST-LINK TCP server so they
can use the SWD/ST-LINK link concurrently. You can step through code in
STM32CubeIDE while CubeMonitor live-plots your variables.

---

## Why it's needed

STM32CubeMonitor can talk to an ST-LINK two ways:

| Protocol | Behavior | Can share the probe? |
|----------|----------|----------------------|
| `p2p` (default) | Direct, exclusive connection to the probe. Highest data rate. | **No** — locks out every other app |
| `tcp` | Goes through **ST-LINK server** (`stLinkServer`), which arbitrates access. | **Yes** |

Shared mode = run CubeMonitor in `tcp` mode **and** tell the IDE to use the shared
ST-LINK. There is a performance cost to `tcp`, so only use it when you actually
need both tools at once.

> **J-Link probes support `p2p` only** — shared mode is an ST-LINK feature.

---

## Step 1 — Confirm ST-LINK server is installed

`tcp` mode needs `stLinkServer` on the host. It ships with STM32CubeIDE and
STM32CubeProgrammer, so if you have either it's almost certainly present.

```bash
# Linux: typical install location
ls /opt/st/stm32cubeide*/plugins/com.st.stm32cube.ide.mcu.externaltools.stlink-gdb-server*/ 2>/dev/null
which stlink-gdb-server 2>/dev/null
```

On Windows it installs under `C:\Program Files\STMicroelectronics\stlink_server\`.
If it's missing, install the **ST-LINK server** add-on from STM32CubeProgrammer.

## Step 2 — Switch CubeMonitor to `tcp` mode

CubeMonitor's protocol is set in its `settings.js` file. Edit it, then restart —
the change only takes effect at the next startup.

`settings.js` location:

| OS | Path |
|----|------|
| Windows / Linux | `~/.STMicroelectronics/stm32cubemonitor/settings.js` |
| macOS | `~/Library/stm32cubemonitor/settings.js` |

Find the `connectionType` entry and set it to `tcp`:

```js
// Protocol to use by stLinkDriver to communicate with the probe:
// - "tcp" to pass through to tcp-server assuming the stLinkServer is installed on the host
// - "p2p" connect directly to the probe (default)
connectionType: "tcp",
```

Save the file and **fully close and reopen STM32CubeMonitor**.

## Step 3 — Enable shared ST-LINK in STM32CubeIDE

Tell the debugger to share the probe instead of taking it exclusively.

1. **Run** > **Debug Configurations...**
2. Select your debug configuration, open the **Debugger** tab.
3. Check **Enable shared ST-LINK**.
4. **Apply**, then **Debug** to start the session.

## Step 4 — Start the CubeMonitor acquisition

With the IDE debug session running, start acquisition in CubeMonitor as usual
(press the **start/play** button on the acquisition node, then open the
dashboard).

Success signal: in the CubeMonitor flow the probe shows **`tcp connected`**, and
both the IDE debugger and the CubeMonitor charts update without either one
dropping the link.

---

## Other IDEs (same idea, different switch)

| IDE | Where to enable shared mode |
|-----|------------------------------|
| **STM32CubeIDE** | Debug Configurations > Debugger tab > **Enable shared ST-LINK** |
| **IAR Embedded Workbench** | Project > Options > Debugger > **Extra Options** > add `--stlink_use_server` |
| **Keil µVision** | Project > Options for Target > Debug tab > ST-Link settings > Debug adapter = **Shareable ST-Link** |

---

## If something fails

- **CubeMonitor still won't connect while the IDE is debugging** — `connectionType`
  is still `p2p`, or CubeMonitor wasn't restarted after the edit. Re-check Step 2
  and restart the app completely.
- **IDE debug fails to start with CubeMonitor open** — **Enable shared ST-LINK**
  isn't ticked in the Debugger tab (Step 3), or `stLinkServer` isn't installed
  (Step 1).
- **It connects but acquisition is laggy / drops samples** — that's the `tcp`
  performance penalty. Reduce the sampling frequency, or switch back to
  `p2p` when you don't need the IDE attached.
- **Using a J-Link** — shared mode isn't available; J-Link is `p2p`-only. Use an
  ST-LINK for simultaneous debug + monitor.

---

## References

- [STM32CubeMonitor wiki — How to configure shared mode](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_configure_shared_mode)
- [STM32CubeMonitor wiki — How to change general settings](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_change_general_settings)
