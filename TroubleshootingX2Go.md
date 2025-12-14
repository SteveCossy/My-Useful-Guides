# Troubleshooting X2Go X11 Forwarding Issues

This guide addresses the scenario where `ssh -Y` works in MobaXterm/Windows but fails inside an **X2Go** session on Linux (e.g., `LSTerminal`), resulting in the error:
`qt.qpa.xcb: could not connect to display` or `connect /tmp/.X11-unix/X50: No such file or directory`.

## Phase 1: Establish a Clean Connection
The standard `entry` gateway script may strip X11 forwarding flags. Use the SSH Jump Host (`-J`) feature to tunnel directly to the target machine.

**Command (Run on Local Machine):**
```bash
# -Y: Trusted X11 Forwarding (Critical for modern Qt apps)
# -J: Jump Host (Bypasses the gateway shell script)
ssh -Y -J entry.ecs.vuw.ac.nz username@target-server
```

## Phase 2: Diagnosis (If Phase 1 Fails)
If the connection still fails, X2Go has likely created the forwarding socket but placed it in a non-standard location where SSH cannot find it.

### 1. Check your Display Number
Run this on the **Local Machine** (inside X2Go terminal):
```bash
echo $DISPLAY
# Example Output: :50
```

### 2. Verify the Socket is Missing
SSH expects the socket at `/tmp/.X11-unix/X{N}`. Check if it exists:
```bash
# Replace X50 with your actual display number
ls -la /tmp/.X11-unix/X50
```
If this returns **"No such file or directory"**, proceed to Step 3.

### 3. Locate the Real Socket
X2Go (nxagent) often hides the socket in a private temp directory. Find it using `find` or `ss`.

**Method A: File Search**
```bash
find /tmp -type s -name "X50" 2>/dev/null
# Likely result: /tmp/.x2go-username/C-session-id/X50
```

**Method B: Socket Statistics (If file search fails)**
```bash
# Look for the process listening on X11
ss -lxl | grep X11
```

## Phase 3: The Fix
Once you find the real path (e.g., `/tmp/real/path/to/X50`), create a symbolic link so SSH can find it.

**Command (Run on Local Machine):**
```bash
# 1. Clean up any stale broken link
rm /tmp/.X11-unix/X50

# 2. Link the real socket to the standard location
# Syntax: ln -s [TARGET_REAL_PATH] [LINK_NAME]
ln -s /tmp/.x2go-username/C-session-id/X50 /tmp/.X11-unix/X50
```

## Phase 4: The "Nuclear" Option (Reboot)
If `find` returns nothing, or the socket exists but refuses connection, the filesystem state may be corrupted (often caused by `systemd-tmpfiles` cleaning up `/tmp` while X2Go was running).

1.  **Reboot the Local Machine.**
2.  Reconnect X2Go.
3.  Verify `$DISPLAY` exists.
4.  Connect immediately.

---

*Verified on NetBSD/Linux environment, Dec 2025.*
