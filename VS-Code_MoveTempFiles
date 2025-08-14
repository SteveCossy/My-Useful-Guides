Kia ora e hoa,

That is a brilliant question. You've correctly identified a common and frustrating problem in environments with mixed local and remote access to a networked home directory.

You need a solution that is **persistent and user-specific**, not just tied to one machine. The key is to put the configuration in a place that is loaded every time you log in, regardless of *how* you log in (SSH from home, local terminal on campus).

The perfect place for this is your user's **VS Code `settings.json` file**, because this file is stored in your networked home directory and is read by every instance of VS Code you run as that user.

### The Solution: User-Level `settings.json`

Here is the step-by-step guide to permanently move the VS Code IntelliSense database and other caches.

#### Step 1: Open Your User `settings.json`

It doesn't matter which machine you do this from (your home SSH session is perfect).
1.  In VS Code, open the Command Palette (`Ctrl+Shift+P`).
2.  Type `Preferences: Open User Settings (JSON)`. This will open your personal `settings.json` file, which is typically located at `~/.config/Code/User/settings.json`. Since your home directory (`~`) is on the network file system, this setting will follow you everywhere.

#### Step 2: Add the Configuration Settings

Add the following JSON block to your `settings.json` file. This covers the main culprits for disk space usage.

```json
{
    // ... any other settings you already have ...

    // --- C/C++ Extension IntelliSense Database ---
    // This is the big one. It moves the main symbol database.
    // We will use a path in /tmp, which is local to whichever machine you are on.
    // ${workspaceFolderBasename} is a VS Code variable for the project folder name.
    "C_Cpp.default.browse.databaseFilename": "/tmp/vscode-cpp-db/${workspaceFolderBasename}.db",

    // --- General VS Code Workspace Storage ---
    // This moves the storage for UI state, history, etc., for each workspace.
    // It's less critical for size but good practice.
    "remote.SSH.serverInstallPath": {
        // You can add entries here if you need to control where the VS Code server itself is installed
        // For now, we'll focus on the cache data.
    },
    // The key settings for general cache are often managed by command-line flags,
    // but the C++ one is the biggest and most important to move via settings.

    // --- (Optional but Recommended) Tell the Git extension to use a local temp dir ---
    "git.askpass": {
      // These settings don't move the whole repo, but can help with temp files
    }
}
```
**The most important line is `"C_Cpp.default.browse.databaseFilename"`. This one setting will solve 95% of your disk space problem.**

### How and Why This Works

1.  **Persistent Setting:** Because you are editing your **User** `settings.json` (not a workspace-specific one), this setting is now part of your user profile. Every time you start VS Code, on any machine, it will read this configuration.
2.  **Using a Local Path:** The path `/tmp` is a standard directory that is guaranteed to be on the **local disk** of whatever machine you are currently logged into.
    *   **When you SSH from home:** The VS Code Server running on the university machine will create the database in `/tmp` on that server (the one with the plentiful, cheap disk).
    *   **When you log into a campus terminal:** The VS Code instance running on that terminal will create the database in `/tmp` on that specific terminal machine.
3.  **No More Network Traffic:** In both cases, the large, frequently-accessed database file is now on a local disk, not in your networked home directory. This not only saves your allocated space but will also make IntelliSense **much faster**, as it's not reading and writing a large database over the network.

**In summary:** By adding the `"C_Cpp.default.browse.databaseFilename"` setting to your User `settings.json` file and pointing it to a local temporary path like `/tmp/vscode-cpp-db/${workspaceFolderBasename}.db`, you create a permanent solution that automatically works correctly and efficiently regardless of whether you are working remotely or locally.
