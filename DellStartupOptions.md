Ah, the "Press F1 to Continue" curse. Dell Optiplex BIOSes are notoriously strict about this because they treat a missing thermal sensor as a critical safety risk (fearing the fans won't know how fast to spin, leading to overheating).

Since you have to walk over there anyway to press F1 this time, here is what you can try **while you are sitting in front of it** to stop it from happening on the next reboot.

### 1. The BIOS Setting (The "Maybe" Fix)
Dell BIOS versions vary, but look for this setting. It works on about 50% of the Optiplex range for this specific error:

1.  Mash **F2** to enter the BIOS Setup.
2.  Go to **POST Behavior**.
3.  Look for **Keyboard/POST Errors** (or just "POST Errors").
4.  Change it from "Halt on All Errors" to **"Continue on Errors"** (or "Report, but Continue").
    *   *Note:* Often, Dell only allows ignoring *Keyboard* errors, not hardware sensor errors. If "Keyboard" is the only option, this software fix won't work.

### 2. The Hardware "Hack" (The Real Fix)
If the BIOS refuses to ignore the error, you have to trick the motherboard.

The Optiplex 990 usually expects the thermal sensor to be connected to the **Front I/O Panel** or a dedicated 2-pin header near the front of the motherboard.

*   **If you can't find the sensor:** The motherboard is looking for a resistance value (usually a thermistor). If the circuit is open (infinite resistance), it throws the error.
*   **The Trick:** If you have a spare **10k Ohm resistor** lying around in a lab drawer, you can shove the legs into the 2-pin thermal sensor header on the motherboard. The BIOS will think it's a pleasant 25°C in the room and boot happily forever.

### 3. Intel AMT (The "Save your legs next time" Fix)
Since it's an Optiplex 990, it has **Intel vPro / AMT** hardware built-in.
While you are at the physical console:
1.  Press **Ctrl+P** during boot (at the Intel ME screen).
2.  Enable AMT and set a password.
3.  If you configure this, you can use a tool like "MeshCommander" (open source) on another PC to remotely access the **hardware KVM**.
4.  This lets you see the BIOS screen and press F1 remotely if this ever happens again.

**Summary:** Try the BIOS "POST Behavior" setting first. If that fails, the machine will simply refuse to boot unattended until that circuit is closed (either by finding the sensor or bridging it with a resistor).
