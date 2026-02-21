# SimGliderControl - Assembly Instructions

## Step 3: Wiring and Calibration

### Mounting and Calibrating Hall Effect Sensors

### Required Parts
- 3× Hall effect sensors AS5600 Encoder (23mm × 23mm) with magnet (4mm × 2mm)
- 1× Arduino UNO or compatible
- 10× M3 button head screws 5mm length
- Wire for connections (recommend different colors for VCC, GND, and signal)
- 12× M4 nuts (for enclosure mounting)
- 4× M3 nuts (for enclosure attachment)
- 4× M3 screws 15mm length

### Software Required
- **Mobiflight Software** - Download and install from https://www.mobiflight.com/en/download.html
- **Getting Started Guide** - https://docs.mobiflight.com/getting-started/ (recommended for beginners)

---

### 3.1 Hall Sensor Operating Mode

The Hall sensors can operate in two modes: digital output or analog output. For Arduino with Mobiflight firmware, we use the **analog variant**.

**Technical specifications:**
- The sensor provides absolute analog values over the full rotation angle 0-360°
- Connected to pins A0-A2 on Arduino with 10-bit AD converter
- With ~120° rotation angle, this results in approximately 210-220 steps resolution
- Over the ~180mm control travel, resolution is less than 0.9mm

**⚠️ CRITICAL - Power Supply:**
The sensor is powered from the Arduino's **GND and 3.3V pins** (NOT 5V!)

This reduces the ~1024 possible values at the analog port to ~675. With 120° rotation angle, this is further reduced by 1/3 ≈ 220 steps.

---

### 3.2 Preparing Hall Effect Sensors

![Step 3 Sensor Preparation](images/s3.jpg)

**For analog operation:**
The GPO and VCC pins must be bridged, either as shown in the image or later via the pin headers.

**Soldering:**
Both pin headers must be soldered on the **back side** of the sensor board.

**Magnet installation:**
1. Press the magnet into the magnet holder - it should not fall out
2. If loose, add a drop of glue
3. **⚠️ Do not lose the magnet!** They have a special orientation that cannot be replaced with standard magnets

![Step 3 Magnet Holder](images/s2-9.jpg)

---

### 3.3 Mount Magnet Holders to Shafts

Press the magnet holders firmly onto the Allen screw head of each shaft. They should not fall off by themselves and must rotate freely with the shaft.

**Test fit each Hall sensor:**
- Sensors must not wobble
- Must sit flat on all 4 screw posts
- Magnet must not be pinched
- Gap should not be too large (ideally only 1-2 sheets of paper between chip and magnet)

Correct function will be verified during calibration.

---

### 3.4 Arduino Installation and Initial Wiring

![Step 3 Arduino Mounting](images/s3-1.jpg)

1. Mount the Arduino to one of the two mounting points in the enclosure using M3 screws
2. **Initial wiring for first sensor (A0):**
   - VCC → Arduino 3.3V pin
   - GND → Arduino GND pin  
   - OUT → Arduino A0 pin

![Step 3 Wiring Detail 1](images/s3-3.jpg)
![Step 3 Wiring Detail 2](images/s3-4.jpg)

---

### 3.5 Mobiflight Firmware Setup

**Connect Arduino to PC and configure:**

1. **Start Mobiflight** and click "Mobiflight Modules"
2. The Arduino should appear in the module list as "Compatible" or "Arduino UNO"
3. Click **"Upload/Update Firmware"** at the bottom - the Arduino is then ready as a module
4. Right-click on the module and **add 3 Analog Ports** (A0, A1, A2)
5. Set **Sensitivity to High** (maximum resolution)
6. Use names: **FLAPS**, **SPOILERS**, **TRIM**
7. For minimal setup, you can also add any digital "Button" input
8. Click **"Upload config"**
9. **"Save"** module config to a file (*.mfmc)

**Configuration files:**
A Mobiflight setup consists of 2 files:
- Module Config (*.mfmc) - defines the hardware pins
- Mapping (*.mcc) - maps module ports to simulator variables

**Template files included in repository:**
- `GliderCtr.mcc` - Mapping configuration
- `MobiFlightUnoGliderCtr.mfmc` - Module configuration with MSFS SimVars

The included Mobiflight config is mapped to a different hardware ID. On first connection, you can change this via a dialog. Then save both configs.
- Every change to the module requires a new upload and save (mfmc)
- Every change to the mapping requires saving the mcc file

![Step 3 Mobiflight Setup 1](images/s2-10.jpg)
![Step 3 Mobiflight Setup 2](images/s3a.jpg)

---

### 3.6 Sensor Orientation and Calibration

For correct sensor orientation, we need a configured Arduino module and Mobiflight.

**Initial sensor test:**
1. Connect the first Hall sensor and loosely insert it into a slot (don't screw down yet)
2. In Mobiflight: **Extra > Settings** - enable **Logging and Debug**
3. In the log, you should see values from the connected input that change when you move the carriage

**⚠️ If no values appear:** Check all wiring and settings

![Step 3 Calibration Values](images/s2-11.jpg)

**Example - Correct values:**
Range: 170 to 374 = 204 steps

**Critical requirement:**
Ensure that the values **do not roll over through 0** over the entire control travel.
- ❌ **Wrong:** Value1 = 600 and Value2 = 150 (rolls through zero)
- ✅ **Correct:** Minimum value must be below 400, maximum value approximately 200 steps higher

The value must later be converted via a formula to the required MSFS range of -16383 to +16383. **If the value rolls through 0, the formula will not work.**

---

### 3.7 Setting Correct Sensor Orientation

![Step 3 Sensor Orientation](images/s2-12.jpg)

1. **Rotate the sensor 90° and test again** until values increment correctly (no rollover through zero)
2. Once correct orientation is found, **fix the sensor with 2 short M3 screws**
3. **Repeat this process for all three sensors**

**Note:** In this example, all sensors happened to have identical orientation by chance. Yours may differ.

---

### 3.8 Final Wiring and Formula Configuration

Connect all sensors to the Arduino:
- Sensor 1 (FLAPS) → A0
- Sensor 2 (SPOILERS) → A1  
- Sensor 3 (TRIM) → A2

**⚠️ Do not screw on the enclosure yet!**

**Start Mobiflight again.** Now the correct values for each sensor must be entered into the formula in the Mobiflight mapping.

---

### 3.9 Calibration Formula Setup

**For each sensor:**
1. Move the carriage back and forth
2. Note the **minimum** and **maximum** values
3. Calculate the **number of steps** (max - min)
4. Enter the min value and number of steps correctly into the formula

**Formula notation:**
Mobiflight uses Polish notation (reverse notation).

---

### 3.10 Example Formulas

**Example for FLAPS:**
- Range: 170 to 374 = 204 steps
- Formula: `@ 170 - 204 / 32766 * 16383 - (>K:AXIS_FLAPS_SET)`
- Explanation: @ - 170 / 204 * 32766 - 16383

**Example for SPOILERS:**
- Range: 67 to 290 = 223 steps
- Formula: `@ 67 - 223 / 16383 * (>K:AXIS_SPOILER_SET)`

**Example for TRIM:**
- Range: 211 to 432 = 221 steps
- Formula: `@ 211 - 221 / 32766 * 16383 - -16383 max 16383 min (>K:ELEVATOR_TRIM_SET)`

**Where:**
- `@` = the analog value at the input
- First number = your minimum value
- Second number = your step count (max - min)
- Rest = MSFS command

**This calibration should only be necessary once.**

---

### 3.11 Verification Before Final Assembly

Before screwing on the enclosure, verify the function multiple times. The Hall sensors are so accurate that values at the endpoints should vary by a maximum of 2-3 steps.

**If variation is larger:**
- Check magnet holder alignment
- Verify distance to Hall sensor
- Ideally, only 1-2 sheets of paper should fit between chip and magnet

---

### 3.12 Enclosure Assembly

![Step 3 Enclosure Nuts 1](images/s3-5.jpg)

**Insert nuts:**
1. Insert 12× M4 nuts into the holders on both sides

![Step 3 Enclosure Nuts 2](images/s3-6.jpg)

2. Push them in so they sit centered in the screw holes
3. Insert 4× M3 nuts for enclosure attachment into the mount

**Final assembly:**
1. Screw the enclosure on with 4× M3 15mm screws
2. **Do not pinch any cables!**
3. Depending on Arduino installation, ensure correct wiring at Arduino
4. If using additional switches and LEDs on the enclosure, connect these now

**Note on additional inputs:**
- Switches in Mobiflight are connected via GND to any I/O pin
- What additional functions you need depends on your desired configuration, the aircraft, and available variables
- This is a deep dive into the thousands of possible SimVars from Mobiflight to MSFS and other simulators

---

### 3.13 Quality Check Before Proceeding

Before moving to Step 4, verify:
- ☑ All sensors respond correctly in Mobiflight log
- ☑ No value rollover through zero
- ☑ Formulas correctly configured for all three axes
- ☑ Values stable within 2-3 steps at endpoints
- ☑ Full travel range works smoothly
- ☑ All wiring secure and not pinched
- ☑ Enclosure mounted properly

---

**Next Step:** Once wiring and calibration are complete and verified, proceed to [Step 4: Front Plates Assembly](ASSEMBLY_STEP4.md)
