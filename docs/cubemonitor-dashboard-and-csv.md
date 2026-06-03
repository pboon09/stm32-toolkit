# Customize the STM32CubeMonitor Dashboard and Export Data to CSV

The STM32CubeMonitor dashboard is a Node-RED flow, so you build it by wiring nodes
together in design mode and then view the result in dashboard mode. This guide
shows how to add widgets such as a gauge, arrange them, and export the acquired
data to a CSV file for Excel or MATLAB. Nothing you change in design mode takes
effect until you press the **DEPLOY** button.

| Mode | What it does | How to open it |
|------|--------------|----------------|
| Design mode | You build the flow with probes, variables, processing, and widgets. | It is the default editor view. |
| Dashboard mode | You watch the live data and pick which variables to read or write. | Click the **DASHBOARD** button or open `http://127.0.0.1:1880/ui` in a browser. |

## Part A, customize the dashboard

### Step 1 — Add a gauge

A gauge needs one variable at a slow update rate, so you put a Single Value node
between the processing node and the gauge.

1. Switch to design mode.
2. Drag a **Single Value** node from the palette and connect it to the output of the **processing** node.
3. Open the Single Value node, choose the variable you want to show, and click **Done**.
4. Drag a **gauge** node from the dashboard section of the palette and connect it to the Single Value node.
5. Open the gauge node, set the **Label**, **Units**, and **Range**, and click **Done**.
6. Press **DEPLOY**.

The gauge now appears in dashboard mode and updates while you acquire. The same
pattern works for the other widgets.

### Step 2 — Configure the chart

Click the **chart** node in design mode, set the chart type, the curve type, and
the group it belongs to, then press **DEPLOY**. In the dashboard the chart toolbar
gives you Brush to drag and zoom into a region, Zoom to use the mouse wheel, Show
Points, Show All, clickable legends to hide a series, and an IMPORT DATA button to
reload a file you logged earlier.

### Step 3 — Lay out the widgets

1. Click the **dashboard** icon at the top-right of the editor.
2. Open the **Layout** tab, where the groups and tabs are listed.
3. Click the **layout** button on the group you want to change, then drag the widgets into the size and position you want.
4. Press **DEPLOY**.

You can also save a whole dashboard with **Menu** > **Export** and load one with
**Menu** > **Import**. Start from an empty workspace before importing so you do not
create duplicate nodes.

## Part B, export data to CSV

The simplest way is the Log option built into the processing node, with no flow
editing.

### Step 1 — Turn on logging in the processing node

1. Open the **processing** node in design mode.
2. Set **Log option** to **Log all values** for every sample, or **Log only changes** for smaller files.
3. Set **Log Format** to **csv (multiple columns)**, which gives one column per variable with a timestamp and is the best choice for a spreadsheet. The single column option and the stcm format are also available.
4. Press **DEPLOY**.

The csv (multiple columns) option needs STM32CubeMonitor version 1.7.0 or newer.

### Step 2 — Find the file

Run an acquisition. CubeMonitor writes a new file every time you start acquisition
on a variable group, named like `Log_groupname_timestamp.csv`. It goes to the log
folder set by the `logpath` value, which defaults to the `LOGPATH` environment
variable or to a folder named `log` in your home directory. You can change the
folder with the `logpath` value in `settings.js`. Open the file in Excel, MATLAB,
or Python.

| Operating system | settings.js location |
|------------------|----------------------|
| Windows | `%USERPROFILE%\.STMicroelectronics\stm32cubemonitor\settings.js` (i.e. `C:\Users\<you>\.STMicroelectronics\stm32cubemonitor\settings.js`) |
| Linux | `~/.STMicroelectronics/stm32cubemonitor/settings.js` |
| macOS | `~/Library/stm32cubemonitor/settings.js` |

### Step 3 — Custom CSV layout with a flow (optional)

If the built-in format does not fit your needs, CubeMonitor can build the CSV with
a flow instead. You can convert an existing `.stcm` log into `.csv` with ST's
"Group variables in a .csv file" subflow, or log straight to `.csv` during
acquisition with ST's direct-CSV flow. Both flows and their import steps are on the
ST wiki page linked below.

## If something fails

- If no CSV appears after acquiring, the **Log option** is still set to **No log** or you did not press **DEPLOY**, so check Step 1.
- If you cannot find the file, look in the `logpath` folder, which defaults to a folder named `log` in your home directory, and set an explicit path in `settings.js` if you are unsure.
- If only the single column option is offered, your CubeMonitor is older than 1.7.0, so update it or use the flow in Step 3.
- If a widget change does not show, you did not press **DEPLOY** or you are editing a leftover hidden node, so remove stray nodes and deploy again.

## References

- [STM32CubeMonitor wiki, Design and dashboard modes](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:Design_and_dashboard_modes)
- [STM32CubeMonitor wiki, How to send data to a gauge](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_send_data_to_a_gauge)
- [STM32CubeMonitor wiki, How to configure the chart](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_configure_the_chart)
- [STM32CubeMonitor wiki, How to record and replay data](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_record_and_replay_data)
- [STM32CubeMonitor wiki, How to log data in a .csv file](https://wiki.st.com/stm32mcu/wiki/STM32CubeMonitor:How_to_log_data_in_a_.csv_file)
