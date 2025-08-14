Kia ora e hoa,

It's a frustrating but necessary part of development when a tool becomes unstable. Building a script to automate the "nuke it from orbit and start again" process is a very wise move.

Your analysis of the `ps` output is excellent, and your approach is very close to perfect. Let's turn that into a robust Bash script.

### Analysis of Your Plan

> "First we need to find the PIDs... There should be three..."
> "To kill the simulator, I need to seriously kill the first and third processes"

This is a brilliant observation. You've correctly identified the three key Java processes launched by Cooja:
1.  **The Gradle Wrapper:** The first process (`2414128`) is the lightweight `gradlew` wrapper. It's the entry point.
2.  **The Gradle Daemon:** The second process (`2414172`) is the main Gradle daemon. It's the build engine.
3.  **The Cooja GUI:** The third process (`2414443`) is the actual Cooja simulator application. This is the one that's likely crashing or hanging.

Your plan to `kill -s KILL` (which is the same as `kill -9`) the first and third processes is a solid strategy. Killing the Gradle wrapper and the Cooja GUI will usually cause the Gradle daemon to shut down cleanly or be orphaned, which is fine.

### The Bash Script

Here is a script that automates this entire process. It's designed to be safe and to avoid accidentally killing the wrong processes.

```bash
#!/bin/bash

# A script to safely find and terminate all Java processes
# related to the Contiki-NG Cooja simulator for the current user.

echo "--- Finding Cooja Simulator Java Processes ---"

# We use pgrep for a safer and cleaner way to find the PIDs.
# It's better than parsing the output of 'ps'.

# Find the PID of the Gradle Wrapper. We look for a Java process
# running the specific 'gradle-wrapper.jar'.
GRADLE_WRAPPER_PID=$(pgrep -f "java.*gradle-wrapper.jar.*run")

# Find the PID of the Cooja GUI application. We look for a Java
# process running 'org.contikios.cooja.Main'.
COOJA_GUI_PID=$(pgrep -f "java.*org.contikios.cooja.Main")

# --- Safety Check ---
# Check if we actually found the PIDs before trying to kill them.

if [ -z "$GRADLE_WRAPPER_PID" ] && [ -z "$COOJA_GUI_PID" ]; then
    echo "No running Cooja processes found. Exiting."
    exit 0
fi


# --- Termination ---
# We kill the processes in a specific order: GUI first, then the wrapper.

if [ -n "$COOJA_GUI_PID" ]; then
    echo "Attempting to kill Cooja GUI (PID: $COOJA_GUI_PID)..."
    kill -9 "$COOJA_GUI_PID"
    if [ $? -eq 0 ]; then
        echo "Successfully killed Cooja GUI."
    else
        echo "Failed to kill Cooja GUI (PID: $COOJA_GUI_PID). It might already be gone."
    fi
else
    echo "Cooja GUI process not found."
fi

echo "" # Add a blank line for readability

if [ -n "$GRADLE_WRAPPER_PID" ]; then
    echo "Attempting to kill Gradle Wrapper (PID: $GRADLE_WRAPPER_PID)..."
    kill -9 "$GRADLE_WRAPPER_PID"
    if [ $? -eq 0 ]; then
        echo "Successfully killed Gradle Wrapper."
    else
        echo "Failed to kill Gradle Wrapper (PID: $GRADLE_WRAPPER_PID). It might already be gone."
    fi
else
    echo "Gradle Wrapper process not found."
fi

echo ""
echo "--- Cooja cleanup complete. ---"

# Optional: Add the command to restart the simulator right here
# echo "Restarting Cooja..."
# cd /home/stevecos/contiki-ng/tools/cooja && ./gradlew run
```

### How to Use the Script

1.  Save the code above into a new file, for example, `restart_cooja.sh`.
2.  Make the script executable: `chmod +x restart_cooja.sh`.
3.  Run the script whenever Cooja crashes: `./restart_cooja.sh`.

### Why This Script is Better and Safer

*   **No `grep`:** It avoids the classic trap of `ps -ef | grep java` which also finds the `grep` command itself.
*   **`pgrep -f`:** This is the standard, modern tool for this job. It searches the full command line of all running processes for a pattern. It's much safer than parsing text.
*   **Specificity:** We are searching for very specific strings (`gradle-wrapper.jar` and `org.contikios.cooja.Main`) that are unique to these processes. This dramatically reduces the risk of accidentally killing some other unrelated Java process you might be running.
*   **Safety Checks:** The script checks if the PIDs were actually found (`if [ -n "$VAR" ]`) before it tries to `kill` them. This prevents errors if you run the script when Cooja isn't running.
*   **Clear Output:** The `echo` statements tell you exactly what the script is doing, which is great for peace of mind.

This script should be a reliable tool to get you back to your `mahi nui` faster after a crash.
