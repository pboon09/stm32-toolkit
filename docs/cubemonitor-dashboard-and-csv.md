# Customize the STM32CubeMonitor Dashboard and Export Data to CSV

STM32CubeMonitor's dashboard is a Node-RED flow: you wire **nodes** together in
**design mode** and view the result in **dashboard mode**. This guide covers the
two things people most often want beyond the default chart — **building a custom
dashboard** (adding gauges, arranging widgets) and **exporting acquired data to
CSV** for Excel / MATLAB.

The two modes:

| Mode | What it's for | How to open |
|------|---------------|-------------|
| **Design** | Build the flow: probes, variables, processing, widgets | the default editor view |
| **Dashboard** | Watch the live data, pick variables, write to target | **DASHBOARD** button, or browse to `http://127.0.0.1:1880/ui` |

> Nothing you change in design mode takes effect until you press **DEPLOY**.

---

## Part A — Customize the dashboard

### Step 1 — Add a gauge (worked example)

A gauge needs a single variable at a low update rate, so you insert a **Single
Value** filter between the processing node and the gauge.

1. Switch to **design mode**.
2. From the palette, drag a **Single Value** node into the flow and connect it to
   the **processing** node's output.
3. Open the Single Value node and select the variable you want to show. Click **Done**.
4. From the palette's **dashboard** section, drag a **gauge** node and connect it
   to the Single Value node.
5. Open the gauge node and set **Label**, **Units**, **Range**, and (optionally)
   sector sizes. Click **Done**.
6. Press **DEPLOY**.

The gauge now appears in dashboard mode and updates during acquisition. The same
pattern works for other dashboard widgets (text, chart, etc.).

### Step 2 — Configure the chart

Select the **chart** node in design mode and set chart type (line/bar), curve
type, and which group it belongs to. In the dashboard, the chart toolbar gives
you **Brush** (drag to zoom a region), **Zoom** (wheel to zoom, drag to pan),
**Show Points**, **Show All**, clickable legends to hide/show series, and an
**IMPORT DATA** button to reload a previously logged file. Press **DEPLOY** after
any change.

### Step 3 — Lay out the widgets

Arrange where each widget sits on the page using the layout editor.

1. Click the **dashboard** icon (top-right of the editor).
2. Open the **Layout** tab. Groups and tabs are listed here.
3. Click the **layout** button on the group you want to rearrange, then drag the
   widgets into the positions / sizes you want.
4. Press **DEPLOY**.

> You can save and share an entire dashboard: **Menu** > **Export** writes the flow
> as JSON; **Menu** > **Import** loads one. Start from an empty workspace before
> importing to avoid duplicate-node conflicts.

---

## Part B — Export data to CSV

The simplest path is the processing node's built-in **Log option** — no flow
editing required.

### Step 1 — Turn on logging in the processing node

1. In design mode, open the **processing** node.
2. Set **Log option**:
   - **No log** — default, nothing written
   - **Log all values** — every sample read from the target
   - **Log only changes** — only when a variable changes (smaller files)
3. Set **Log Format** to one of:
   - **csv (one column)** — all data in a single column (raw)
   - **csv (multiple columns)** — one column per variable, with a timestamp (best
     for spreadsheets; needs CubeMonitor **1.7.0+**)
   - **stcm** — CubeMonitor's own format, re-importable into a chart
4. Press **DEPLOY**.

### Step 2 — Acquire, then find the file

Run an acquisition. A **new log file is created each time you start acquisition**
on a variable group, named:

```
Log_<groupname>_<timestamp>.csv
```

It's written to the log directory (`logpath`). The default is the `$LOGPATH`
environment variable, or `~/log` if that's unset. You can change it via the
`logpath` entry in `settings.js`:

| OS | `settings.js` path |
|----|--------------------|
| Windows / Linux | `~/.STMicroelectronics/stm32cubemonitor/settings.js` |
| macOS | `~/Library/stm32cubemonitor/settings.js` |

Open the resulting `.csv` directly in Excel, MATLAB, Python, etc.

### Step 3 (optional) — Custom CSV layout via a flow

If the built-in format doesn't match what you need, CubeMonitor can build CSV
through a flow instead:

- **Convert an existing `.stcm` log to `.csv`** — import ST's "Group variables in
  a .csv file" subflow, point it at the `.stcm` file, set the column header and
  the **Single Time** option (one shared timestamp vs. one per variable), then
  press the **Start** node. The `.csv` is written next to the `.stcm`.
- **Log straight to `.csv` during acquisition** — import ST's direct-CSV flow, add
  one **Select .csv variables** subflow per variable, set the absolute output path
  in the **file** node, and the file updates live as you acquire.

Both flows and their exact JSON are on the ST wiki page linked below.

> **Version note:** CSV export arrived in CubeMonitor **1.4.0**; the layout was
> improved in **1.6.0** (timestamp first, time starting at 0); **multi-column**
> CSV built from the export UI arrived in **1.7.0**. Update CubeMonitor if you
> don't see these options.

---

## If something fails

- **No CSV after acquiring** — **Log option** is still **No log**, or you didn't
  **DEPLOY** after changing it. Re-check Step 1.
- **Can't find the file** — look in `logpath` (default `~/log` or `$LOGPATH`).
  Set an explicit path in `settings.js` and restart CubeMonitor if unsure.
- **Only "csv (one column)" is offered** — your CubeMonitor predates multi-column
  CSV (1.7.0). Update the tool, or use the custom flow in Step 3.
- **Widget changes don't show** — you didn't press **DEPLOY**, or you're editing a
  hidden/duplicate node left over from an import. Remove stray nodes and redeploy.

---

## References

- [STM32CubeMonitor wiki — Design and dashboard modes](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:Design_and_dashboard_modes)
- [STM32CubeMonitor wiki — How to send data to a gauge](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_send_data_to_a_gauge)
- [STM32CubeMonitor wiki — How to configure the chart](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_configure_the_chart)
- [STM32CubeMonitor wiki — How to record and replay data](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_record_and_replay_data)
- [STM32CubeMonitor wiki — How to log data in a .csv file](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_log_data_in_a_.csv_file)
