# Tutorial: Running Google Antigravity on Pixel 10 Pro (Debian Terminal)

> **Validated on:** Pixel 10 Pro Fold (Android 16, Jan 2026 Security Patch).

This guide covers how to install, authenticate, and interact with Google Antigravity on a Pixel 10 Pro (or Fold) using the Android 16 Terminal app.

It is a bit clunky, but it's the kind of "clunky" that feels reliable once you get the muscle memory down.

## The Challenge

The Android Terminal runs in an isolated Virtual Machine (AVF). This causes two main issues:
1.  **Authentication:** The browser on Android cannot see localhost inside the Linux VM.
2.  **Clipboard:** The Linux GUI cannot "see" the Android clipboard directly.

## The Solution

We use a manual script to bridge authentication and a clipboard tool (`wl-clipboard`) to pass text from Android to the Linux GUI.

## Prerequisites

*   Pixel 10 / 10 Pro / Fold (Android 16)
*   Android Terminal enabled in Settings.
*   A Bluetooth keyboard/mouse is highly recommended.

## Step 1: Prepare the Linux Environment

Open the Terminal app. We need to install standard tools and the clipboard manager.

1.  Update the package lists and upgrade existing packages:
    ```bash
    sudo apt update -y && sudo apt upgrade -y
    ```

2.  Install GPG (for the installer) and `wl-clipboard` (for copy/paste support):
    ```bash
    sudo apt install gpg wl-clipboard
    ```

## Step 2: Install Antigravity

1.  Navigate to `https://antigravity.google/download` on your phone.
2.  Follow the standard Linux installation instructions provided on the page.
3.  Once the installation completes, run a final update:
    ```bash
    sudo apt update && sudo apt upgrade
    ```

## Step 3: Launch the GUI

Start the application by typing:
```bash
antigravity
```

*   **To see the interface:** Tap the Display Icon in the top right corner of the Terminal app window (looks like a small monitor).
*   **Important:** The Terminal will request permission to open ports several times. Approve **all** port requests.

## Step 4: The Authentication Workaround

Because the Linux VM is isolated, clicking "Sign In" will fail to redirect back to the app. You must bridge the connection manually.

1.  **Trigger Sign-in:** In the Antigravity GUI, click "Sign in with Google."
2.  **Approve the Port:** Terminal will ask to expose a new port. Approve it and write down the port number (e.g., `35123`).

### A. Create the Helper Script

Switch to the text-based Terminal view and create a script to generate your login URL.

1.  Create/Edit the file:
    ```bash
    nano auth.sh
    ```

2.  Paste the following code:
    ```bash
    #!/bin/bash
    read -p "Enter the port number from the Antigravity log: " port
    CLIENT_ID="1071006060591-tmhssin2h21lcre235vtolojh4g403ep.apps.googleusercontent.com"
    SCOPES="https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcloud-platform%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.email%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fuserinfo.profile%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fcclog%20https%3A%2F%2Fwww.googleapis.com%2Fauth%2Fexperimentsandconfigs"
    REDIRECT_URI="http%3A%2F%2Flocalhost%3A${port}%2Foauth-callback"
    STATE="linux_auth_$(date +%s)"
    AUTH_URL="https://accounts.google.com/o/oauth2/v2/auth?client_id=${CLIENT_ID}&redirect_uri=${REDIRECT_URI}&response_type=code&scope=${SCOPES}&access_type=offline&prompt=consent&state=${STATE}"

    echo "$AUTH_URL"
    ```

3.  Save (**Ctrl+O**) and Exit (**Ctrl+X**).
4.  Make it executable:
    ```bash
    chmod +x auth.sh
    ```

### B. Perform the "Manual Bridge"

1.  Run the script:
    ```bash
    ./auth.sh
    ```
2.  Enter the port number Antigravity requested.
3.  Copy the URL output by the script.
4.  **Login in Chrome:** Open Chrome on Android, paste the URL, and sign in.
5.  **Wait for the Timeout:** Chrome will eventually show `ERR_CONNECTION_REFUSED`. **This is expected.**
6.  **Copy the Failed URL:** Copy the URL from Chrome's address bar (it will contain `?code=...`).
7.  **Bridge with Curl:** Back in your Terminal, run:
    ```bash
    curl "PASTE_THE_FAILED_URL_HERE"
    ```
    > [!IMPORTANT]
    > Ensure you use quotes around the URL!

8.  Check the GUI; you should be signed in.

## Step 5: How to Copy/Paste (Android to Linux)

The Antigravity GUI cannot "see" what you copy on Android. You must "inject" the text into the Linux clipboard.

When you need to paste text (like an API key) from Android into the Antigravity window:

1.  Copy the text on Android as usual.
2.  Switch to the **Terminal view** (not the GUI).
3.  Type `wl-copy` and press **Enter**. (The cursor will hang, waiting for input).
4.  Paste your text into the Terminal (**Ctrl + Shift + V** or Long press -> Paste).
5.  Press **Enter** (to capture the last line).
6.  Press **Ctrl + D** (to tell Linux "End of Input").
7.  Switch back to the Antigravity GUI and press **Ctrl + V**.
