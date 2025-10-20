That's a great approach for comparing changes! Here's a script that should help you automate that process. It will:

1.  Find files containing "hton" in your developed codebase, excluding those with "rpl-lite".
2.  For each found file, it will determine the corresponding path in the original Contiki-NG codebase.
3.  Then, it will perform a `diff -u` between your modified file and the original file.

```bash
#!/bin/bash

# Define your base directories
DEVELOPED_DIR="/home/stevecos/Contiki-ng"
ORIGINAL_DIR="/local/scratch/stevecos/contiki-ng"

echo "Comparing files containing 'hton' (excluding 'rpl-lite') between:"
echo "Developed: $DEVELOPED_DIR"
echo "Original:  $ORIGINAL_DIR"
echo "------------------------------------------------------------------"

# Find files in your developed codebase
find "$DEVELOPED_DIR" -type f -exec grep -lE "hton" {} + | grep -v "rpl-lite" | while read -r DEVELOPED_FILE; do
    # Get the relative path from the developed directory
    RELATIVE_PATH="${DEVELOPED_FILE#$DEVELOPED_DIR/}"

    # Construct the path to the original file
    ORIGINAL_FILE="$ORIGINAL_DIR/$RELATIVE_PATH"

    # Check if the original file exists before trying to diff
    if [ -f "$ORIGINAL_FILE" ]; then
        echo "--- Diffing: $RELATIVE_PATH ---"
        diff -u "$ORIGINAL_FILE" "$DEVELOPED_FILE"
        echo "" # Add a newline for better readability between diffs
    else
        echo "--- WARNING: Original file not found for $RELATIVE_PATH (might be new) ---"
        echo "Developed file: $DEVELOPED_FILE"
        echo "------------------------------------------------------------------"
    fi
done

echo "------------------------------------------------------------------"
echo "Comparison complete."
```

**How to use this script:**

1.  **Save it:** Save the code above into a file, for example, `compare_contiki.sh`.
2.  **Make it executable:** `chmod +x compare_contiki.sh`
3.  **Run it:** `./compare_contiki.sh`

**Explanation:**

*   **`DEVELOPED_DIR` and `ORIGINAL_DIR`**: These variables store the paths to your two Contiki-NG directories. Make sure they are correct.
*   **`find "$DEVELOPED_DIR" -type f -exec grep -lE "hton" {} +`**: This part finds all regular files (`-type f`) within your `DEVELOPED_DIR` and then uses `grep -lE "hton"` to list (`-l`) only the files that contain the string "hton". The `+` makes `grep` run on multiple files at once for efficiency.
*   **`| grep -v "rpl-lite"`**: This pipes the output of the previous `grep` command and filters out any file paths that contain "rpl-lite".
*   **`| while read -r DEVELOPED_FILE; do ... done`**: This loop reads each file path found by the previous commands, one by one, into the `DEVELOPED_FILE` variable.
*   **`RELATIVE_PATH="${DEVELOPED_FILE#$DEVELOPED_DIR/}"`**: This uses shell parameter expansion to remove the `$DEVELOPED_DIR/` prefix from the `DEVELOPED_FILE` path, giving you just the relative path (e.g., `os/net/ipv6/uip-ds6.c`).
*   **`ORIGINAL_FILE="$ORIGINAL_DIR/$RELATIVE_PATH"`**: This constructs the full path to the corresponding file in the original codebase.
*   **`if [ -f "$ORIGINAL_FILE" ]`**: Checks if the original file actually exists. This is important because you might have created new files in your developed codebase that don't have an equivalent in the original.
*   **`diff -u "$ORIGINAL_FILE" "$DEVELOPED_FILE"`**: This performs a unified diff between the original and developed files. Unified diffs are generally easy to read, showing added lines with `+` and removed lines with `-`.
*   **`echo "--- WARNING: ..."`**: If an original file isn't found, it prints a warning.

This script should give you a clear overview of the differences for the files you're interested in!
Let me know if you'd like any adjustments or further modifications to this script! 


I made a few changes to get what I wanted.
#!/bin/bash

# Compare files from development tree with original version
# 20 October 2024

# Define your base directories
DEVELOPED_DIR="/home/stevecos/contiki-ng"
ORIGINAL_DIR="/local/scratch/stevecos/ben/contiki-ng"

echo "Comparing files containing 'hton' (excluding 'rpl-lite') between:"
echo "Developed: $DEVELOPED_DIR"
echo "Original:  $ORIGINAL_DIR"
echo "------------------------------------------------------------------"

# Find files in your developed codebase
find "$DEVELOPED_DIR" -type f -exec grep -lE "hton" {} + | grep -v "rpl-lite" | while read -r DEVELOPED_FILE; do
    # Get the relative path from the developed directory
    RELATIVE_PATH="${DEVELOPED_FILE#$DEVELOPED_DIR/}"

    # Construct the path to the original file
    ORIGINAL_FILE="$ORIGINAL_DIR/$RELATIVE_PATH"

    # Check if the original file exists before trying to diff
    if [ -f "$ORIGINAL_FILE" ]; then
        echo -n "--- Diffing: $RELATIVE_PATH --- "
        if diff -q "$ORIGINAL_FILE" "$DEVELOPED_FILE" >/dev/null; then
           echo "same"
        else
           echo "Showing differences in kdiff3"
           kdiff3 "$ORIGINAL_FILE" "$DEVELOPED_FILE"  > /dev/null 2>&1 &
    #    echo "" # Add a newline for better readability between diffs
         fi
    else
        echo "--- WARNING: Original file not found for $RELATIVE_PATH (might be new) ---"
        echo "Developed file: $DEVELOPED_FILE"
        echo "------------------------------------------------------------------"
    fi
done

echo "------------------------------------------------------------------"
echo "Comparison complete."
[1]   Done                    gedit /home/stevecos/scripts/diff-all.sh > /dev/null 2>&1
[stevecos@bordeaux ~]$

A couple of observations on your changes:

*   **`diff -q "$ORIGINAL_FILE" "$DEVELOPED_FILE" >/dev/null`**: This is perfect for checking if files are identical without printing the actual diff. Its exit status (0 for identical, 1 for different) is what's used by the `if` statement.
*   **`echo "Showing differences in kdiff3"`**: This provides clear feedback to the user about which files are being opened in `kdiff3`.
*   **`kdiff3 "$ORIGINAL_FILE" "$DEVELOPED_FILE" > /dev/null 2>&1 &`**:
    *   `kdiff3 ...`: Launches the `kdiff3` application.
    *   `>/dev/null 2>&1`: Redirects both standard output (`>` `1`) and standard error (`2>`) to `/dev/null`. This prevents `kdiff3` from printing messages to your terminal, keeping the output clean.
    *   `&`: This is crucial! It runs `kdiff3` in the background, allowing the script to continue processing the next file without waiting for you to close `kdiff3` for the current file. This is exactly what you'd want for a batch diff operation.

Your update makes the script much more interactive and user-friendly for this specific task. Ka pai!
