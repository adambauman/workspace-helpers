# Sunshine/Moonlight Ultimate Streaming Setup
Adam J. Bauman (https://github.com/adambauman)

Adapted from xuvvy0's guide: https://www.reddit.com/r/cloudygamer/comments/1coq5f3/stream_with_sunshine_moonlight_in_any_resolution/

Setup self-hosted game streaming with minimal impact to the host PC's "daily driving" functionality. 

### Goals

- Access a high-quality, low-latency game stream on any device within your home
- The host adapts its resolution and refresh rate to match the client's requests
- No annoying management of display settings, etc. when you want to use the host PC directly
- No need for dummy display hardware or weird hacks
- Open the possibility of running the host as a headless game stream server

### Known Issues

- During display swapping the saved position of windows and desktop icons may change. If you can't get a window to show up after streaming is disconnected: press Win+Shift+LeftArrow a few times to move the window between monitor spaces.
- This guide doesn't include HDR instructions: it's pretty easy to enable after you follow this guide, but still in testing.
- If you use Windows Night light you may need to toggle it a couple times after switching to get back to your preferred setting
- Games that have been configured to use a specific display may fail to switch to the virtual adapter. Switch the game's config to automatically select display adapters
- Steam Big Picture refuses to exit and causes Moonlight to get locked into Steam mode: In Moonlight try the Desktop option and close Steam manually.
- Something go really screwed up and now my system is locked on the virtual display: Use another device to access your host's Sunshine management portal, go to the "Troubleshooting" tab, click the "Restart Sunshine" option. If that fails try reconnecting from Moonlight in desktop mode, then disconnecting. Worst case, reboot.

### Document Development Environment

- Host PC: Win11 24H2 x64 Pro, Ryzen 7800x3d, 64GB RAM, nVidia RTX 4090
- Clients: Surface Pro 11th Edition, iPad Pro 11th Gen, Apple TV 4K (2nd Generation), Steam Deck
- Sunshine v2025.822.34814
- Moonlight PC v6.1.0
- Beta: Virtual Driver Control 25.7.23

## Host

This guide assumes you have kept up-to-date with OS and driver updates, and already have Steam installed. We're sticking with a relatively new Microsoft Windows environment for this document, though this sort of thing is definitely possible with a Linux host!

You will be installing a virtual display driver, a multi-monitor profile tool, and the Sunshine streaming host.

> [!WARNING]
> There may be security or privacy risks associated with game stream hosting. Sunshine runs as a service using the local system account, giving it broad access to the OS and your user data. Set a secure username and password during Sunshine setup, don't expose the service's ports outside of a trusted network. Use this guide at your own risk!

### Requirements

- Windows 11 22H2+
- Hardware that meets Sunshine's requirements (see [Sunshine's Readme](https://github.com/LizardByte/Sunshine))
- A 1GbE(Gigabit) Ethernet connection is highly recommended

### Host Configuration

These are a few recommendations for tweaks you can make to the host PC.

#### System Availability

- Option A: Disable System Sleep and Hibernate is the OS power settings. It's okay to allow monitors to sleep or power off.
- Option B: Enable Wake-On-LAN (WOL) and use Moonlight's "Wake Host" or another method to send a magic wake packet to the host.

#### User Logon

- Option A (more secure for a daily driver): Keep your users password protected, leave "lock on inactivity" settings enabled. You will need to select the "Desktop" stream when connecting from a client, then log in before switching to the "Steam Big Picture" stream.
- Option B: Create a streaming-only, non-admin user. Disable any auto-lock settings for this user, then use [https://learn.microsoft.com/en-us/sysinternals/downloads/autologon](Sysinternals' Autologon) to automatically log the streaming user in on system startup. You will need to switch back to your daily-driving user when sitting down at the system.

#### Game Bar
- Open the Game Bar (WIN+G) and disable the "Open Game Bar when I press "XBOX" button" option. Repeat this on any Windows client devices, it's annoying when you're trying to access the Steam Big Picture menu or use Steam's controller-as-mouse mappings.

#### Steam

- Enable Steam Input and install any optional input drivers Steam offers for your controller(s)

### Virtual Display Driver (VDD)

Creates a virtual display port that will be assigned to the Sunshine service. This allows Sunshine to match resolutions and refresh rates with connected clients without being limited by a physical monitor's capabilities. Using the virtual display will also limit annoying changes to your direct-use resolution, scaling, and multi-monitor preferences.

> [!NOTE]
> A commented copy of my vdd_settings.xml is in the Resources directory. Only use it as a reference to get started configuring resolutions and refresh rates for your clients devices.

#### Installation

1. Create a directory in a location available to the entire system. We'll use "C:\game-stream-host" for rest of this document.
1. Download the latest Virtual Driver Control (VDC) app from https://github.com/VirtualDrivers/Virtual-Display-Driver/releases
1. Extract the contents of the release into "C:\game-stream-host\virtual-display-driver"
1. Run "VDD Control.exe", click the "Install Driver" button, wait for the VDC console to display "Successfully connected to the installed driver"

#### Configuration 
1. Run "VDD Control.exe" if it isn't already running
1. Click the "Tools" menu -> XML Editor
    - Select the "Refresh Rates" tab and Add any specific rates your client devices may require
    - Select the "Resolutions" tab and Add the native resolutions of all your client devices. The default refresh of 30Hz is fine for these entries.
    - Click the "Save Changes" button and confirm that you want to close the XML editor
1. Click the "Restart Driver" button

#### Verification

1. Open your Windows Display Settings, you should see a new display listed.
1. Select the new display and verify the "Display Resolution" menu contains your custom resolutions
1. Open the "Advanced display" menu and verify the "Choose refresh rate" menu contains your custom refresh rates

You can now close the Virtual Driver Control (VDC). If you need to adjust the configuration at any time: re-open the VDC, use the XML editor tool, then restart the driver.

### Sunshine

Sunshine will run as a service on the host and facilitate game streaming. This also hosts a management web portal where you will configure the service and connect new devices.

#### Installation

1. Download the newest Sunshine installer from https://github.com/LizardByte/Sunshine/releases/
1. Install Sunshine with the default options, enter a secure username and password when prompted and test that you can log in.
1. (Optional) Lock your host to a static IP address, this way you can quickly access the management portal from a mobile device, laptop, etc. when connecting new devices.
1. (Optional) Set the Sunshine Service to run automatically on system statup:
    1. Open a run window, run ```services.msc```
    1. Double-click "Sunshine Service"
    1. Set "Startup Type" to "Automatic"
    1. Click the OK button to save, then close the Services window.

#### Audio/Video Configuration

1. Run Sunshine, or click its icon in the System Tray and select "Open Sunshine" if it's already running
1. Open the "Troubleshooting" tab, scroll down to the "Logs" section
1. Search through the list display adapters until you find the "VDD by MTT" entry. Copy the "device_id" GUID value and paste it into something like Notepad.
1. Open the "Configuration" tab, open the "Audio/Video" tab, scroll to the "Display Device Id" entry
1. Paste the "device_id" GUID (not including any quotes) we copied earlier into "Display Device Id" box
1. Expand the "Advanced display device options" section and make these changes:
    - Device configuration: Deactivate other displays and activate only the specified display
    - Resolution: Use the resolution provided by the client
    - Refresh rate: Use FPS value provided by the client
    - HDR: Switch on/off the HDR mode as requested by the client
1. Scroll back up a bit and make sure the "Install Steam Audio Drivers" and "Stream Audio" options are selected
1. Click the "Save" button, then the "Apply" button

You're now ready to begin connecting clients!

## Clients
These are general instructions for installing and setting up a Moonlight client. Some clients may not offer the same options and may require tweaking to unlock the best experience.

1. Download and install the newest Moonlight https://github.com/moonlight-stream/moonlight-qt/releases
1. Open Moonlight and click the "+" to add a host
1. Select the host, log into your Sunshine management portal, then add the client's PIN in the "PIN" tab
1. Back to Moonlight, select the newely added host, then hit the Settings (Gear) icon
1. Adjust the following settings:
    - Resolution and FPS: Native, 60 or 120 FPS (if your network and devices can handle it 120FPS will provide the lowest latency)
    - Video bitrate: Set as high as your setup will allow. 150Mbps with modern 5Ghz wireless or Ethernet clients is cake.
    - Display mode: Fullscreen
    - V-Sync: Checked
    - (Initial Testing or Troubleshooting) Show performance stats while streaming: Checked
1. Hit the back arrow

You're now ready to play, if you select the Steam option it should connect to the host and auto-launch Steam in Big Picture mode.

The connection may fail and drop back to the Moonlight UI if the OS is stuck at a user login or lock screen. Use the Desktop option to get past the login screen, then drop out and try Steam again.
