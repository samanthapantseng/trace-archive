# Hand Trail Gesture Tracker

Captures hand movements during conversations between two people and exports them as plottable SVG files.

## How It Works

This system tracks both hands of all people in the camera view, recording their movement trails over time. The trails are then exported as SVG files that can be physically plotted using an AxiDraw plotter.

**Recording Source:**

-   **With live ZED 3D cameras:** Real-time capture of people interacting
-   **With pre-recorded files:** Playback of `.ssrec` recordings (current workflow)

## Complete Workflow

### Step 1: Run the Tracker

```bash
python main.py --rec 'recordings/your_recording.ssrec' --calibration your_calibration
```

**What happens:**

-   Opens a 3D visualization window showing skeleton tracking
-   Records hand movements for **20 seconds** (configurable in `detectors/trail.py` line 30)
-   Tracks both left and right hands of all people in view
-   Automatically exports SVG files when recording completes

**Live camera mode (if you have ZED cameras):**

```bash
python main.py --server server_number --calibration your_calibration
# No --rec flag means live camera feed
```

### Step 2: Generated Files

After 100 seconds, files are automatically saved to:

```
trails/trail_YYYYMMDD_HHMMSS/
├── all_trails_YYYYMMDD_HHMMSS.svg        # Preview of all hands (DO NOT PLOT)
├── person_0_left_YYYYMMDD_HHMMSS.svg     # Individual hand trail (PLOT THIS)
├── person_0_right_YYYYMMDD_HHMMSS.svg    # Individual hand trail (PLOT THIS)
├── person_1_left_YYYYMMDD_HHMMSS.svg     # If multiple people detected
└── person_1_right_YYYYMMDD_HHMMSS.svg    # If multiple people detected
```

### Step 3: Plot the Trails

```bash
./plot_latest.sh
```

**What happens:**

1. Script finds the most recent `trails/trail_*` folder
2. Opens the first individual hand SVG in Inkscape
3. **You manually:** Go to Extensions > iDraw 2.0 Control > Apply
4. Wait for plotting to complete
5. Press **Enter** in the terminal
6. Next file opens automatically
7. Repeat steps 4 for each hand

**Why manual plotting?**  
Inkscape's command-line interface crashes on macOS when trying to automate the iDraw extension. Manual triggering is the only reliable method.

## Output Details

-   **Format:** A5 landscape (210mm × 148mm)
-   **Scale:** Fixed at 800mm height × 1200mm width (seated arm movement range)
-   **Duration:** 20 seconds of recording (adjustable)
-   **One SVG per hand:** Each person's left and right hands get separate files for individual plotting

## Files & Directories

-   **`main.py`** - Main application (live camera or playback mode)
-   **`calibration.py`** - Tool to define floor polygon for filtering
-   **`plot_latest.sh`** - Script to sequentially open SVG files for plotting
-   **`detectors/trail.py`** - Hand tracking logic and SVG export
-   **`recordings/`** - Pre-recorded ZED camera files (`.ssrec`)
-   **`calibrations/`** - Saved calibration polygons (`.json`)
-   **`trails/`** - Exported SVG files organized by timestamp

## Configuration

Edit `detectors/trail.py`:

-   **Line 30:** `self._duration_seconds = 20` - Change recording length
-   **Lines 314-315:** `max_gesture_height_mm` / `max_gesture_width_mm` - Adjust scale

## Tips

-   **Preview first:** Open `all_trails_*.svg` to see the combined visualization before plotting
-   **Multiple people:** Each person gets a unique color and separate SVG files
-   **Fixed scale:** All sessions use the same scale, making gestures comparable
-   **Individual plotting:** Plot each hand separately for cleaner results

## Troubleshooting

### AxiDraw Not Detected ("Failed to connect to iDraw2")

If you see errors like `error open com_port: None` or `Failed to connect to iDraw2`:

1. **Check connections:**
   - AxiDraw is plugged into Mac via USB
   - AxiDraw is powered on
   - Try different USB cable/port

2. **Check USB device:**
   ```bash
   ls -l /dev/cu.*
   ```
   Look for `/dev/cu.usbmodem*` or `/dev/cu.usbserial*`

3. **Restart sequence:**
   - Unplug USB from AxiDraw
   - Power off AxiDraw
   - Wait 10 seconds
   - Power on AxiDraw
   - Reconnect USB cable

4. **Check in Inkscape:**
   - Extensions > iDraw 2.0 Control
   - Look in Setup tab for detected port

5. **Install drivers (if needed):**
   - Download FTDI VCP drivers: https://ftdichip.com/drivers/vcp-drivers/
   - Restart Mac after installation

### SVG Files Empty or Incorrect

- Verify hands were visible during recording in the visualization window
- Check calibration polygon includes the interaction area
- Ensure recording duration completed (20 seconds default)
