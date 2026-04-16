# Kanvas Steam Deck Setup Guide

![IMG_20260409_204111306](https://github.com/user-attachments/assets/b1f428fa-964d-4f28-9b95-6a5fbf0ee555)

For all you portable gaming device fans. This document details the process for setting up the ROBLOX Launcher known as Kanvas to run on a Steam Deck under SteamOS. The instructions in some ways are much similar to what you need to do according to the official Linux instructions, though there are some things which may be done differently, some extra SteamOS specific steps, and some extra tweaks that we'll need to do in the Wine settings to ensure everything works properly. The SteamOS platform is just a modified version of Arch Linux so we can install packages from the Arch repositories, such as the ASP.NET Core Runtimes required to run the Kanvas web server.

Please note that this is not currently supported by the Kanvas developers, so everything in here is provided as-is with no guarantees that anything will be free of issues. Don't send them bug reports for Kanvas issues related to SteamOS as this is just a fun experiment I figured out in my freetime that I want to share with everyone who is interested in playing old Roblox clients on the Steam Deck. If you notice that anything in this documentation is errorenous and in need of correction, then feel free to write it on the Issues page and I will fix it. 

There is a significant limitation with this setup in that each time you install a SteamOS update, you'll need to repeat everything in this guide to reinstall the ASP.NET runtime packages onto your root file system. There does not seem to be any workaround to this other than not installing any updates, which is unadvisable for security reasons. With that being said, let's get started with the initial setup tasks.
# Getting Started
The first thing we'll need to do is enter **Desktop Mode** from Steam's big picture mode. Press the Steam Button and scroll down to **Power** and then select **Switch to Desktop**

Open the Application Launcher menu once in Desktop Mode, move to the **System** group, and launch **Konsole**

A terminal window will launch. This is how we will install the ASP.NET 8.0 Runtime packages. It is strongly recommended that you plug in a USB keyboard for these next steps, though you can also bring up a touchscreen operated on-screen keyboard by pressing **Steam + X**

In the Konsole window, type **passwd** and hit Enter. You'll need to set a root password with which to use sudo commands. If you already have a root password, you may skip this step.

After the root password has been set, type **sudo steamos-readonly disable** to unlock the file system. This step is required to install the ASP.NET runtime packages. With the root file system unlocked, we're now ready to move to the next step.
# Installing Dependencies

**IMPORTANT: Read or else you will get pacman keyring errors!!!**

SteamOS's base image comes with a pacman keyring that does not have any way of verifying packages in the Arch repositories. To fix this, we need to manually initialize the keyring and then populate it with the **holo** keys. Holo is the branch which contains the SteamOS specific packages for Arch based distributions, so we want to initialize the keyring with Holo, though we can still install the key packages for both Holo and Arch. Please follow the guide below to initialize the keyring and load in the relevant GPG keys to get pacman working.

1. Open the pacman configuration file with the command **sudo nano /etc/pacman.conf**
2. Look for the line **SigLevel = Required DatabaseOptional** and change it to **SigLevel = TrustAll** and then press **Ctrl + O** to save the changes, followed by **Ctrl + X** to exit. If you are using the SteamOS on-screen keyboard, you will need to bind one of the buttons to trigger the Ctrl key when pressed as there is no Ctrl button on the on-screen keyboard.
3. After exiting the text editor, run **sudo pacman-key --init** and then **sudo pacman-key --populate holo** followed by **sudo pacman -Sy holo-keyring archlinux-keyring**
4. Open /etc/pacman.conf in nano again with **sudo nano /etc/pacman.conf** and change **SigLevel = TrustAll** back to **SigLevel = Required DatabaseOptional** and then hit **Ctrl + O** to save and **Ctrl + X** to exit.
5. As soon as the keyring is initialized, we can now install the required ASP.NET 8.0 packages for the Kanvas webserver. Run **sudo pacman -Syu** and then **sudo pacman -S aspnet-runtime-8.0** and ASP.NET should now install.

Once ASP.NET 8.0 has been installed, be sure to run **sudo steamos-readonly enable** to re-enable write protection on the root file system. You may now set aside the Konsole window for now, though we will need it again for later.

On the Desktop Mode taskbar, launch the Discover Store and search Lutris while on the Home page. Install the Lutris app that shows in the results, which will set it up using the Flatpak package. This is the officially documented way for running Kanvas on any Linux distribution.

**Note: If the Discover Store download is stalling or produces an error, flatpak may be having some issues. In most cases, this can be fixed by rebooting the system, or turning the Wi-Fi off and back on.**

With Lutris installed, we are now ready to begin setting up Kanvas to run on SteamOS.
# Setting Up Kanvas
You'll need some way of copying Kanvas files to the Steam Deck. You can either use a microSD card in the Steam Deck's built-in SD slot, or a USB storage device which can connect using the USB-C charging port. Refer to the official Kanvas Linux Documentation at https://gitgud.io/playervalley/kanvaslauncherpublic/-/blob/master/docs/Linux.md for setting up Kanvas to run using Proton & Winetricks from the Lutris application. (Credit to sorelight and playervalley for publishing this).

A couple of things to note while following the Lutris setup instructions in the official documentation:
* When creating the winepref folder, it's a good idea to put this somewhere not inside the Kanvas folder. This is where Windows system files are staged for the Wine compatibility later and it can become rather large. I find that putting this in the home folder works good.
* When installing Windows DLLs & applications or installing fonts in winetricks, you'll want to make sure you wait until the winetricks configuration window comes back up to know once either of these steps are finished. The fonts in particular will show many errors stating that a process is being waited on, though these can be ignored. Once the winetricks window reappears, all the required fonts and Windows components are installed.

After following all the official Lutris setup steps, there is one more thing we need to do. This step is required to ensure that newer ROBLOX clients with native controller support don't encounter issues with SteamOS's built in controller button keyboard bindings. Open the menu arrow next to the Wine icon, and select **Wine Control Panel**

In the Wine Control Panel, open the **Game Controllers** settings menu. There should be an **XBOX 360 For Windows** and a **Steam Deck controller** listed under XInput Devices in the Joysticks tab. For each controller, select it's entry and click **Override** to move it to DirectInput Devices. Then select each one again after it is moved to the DirectInput section and select **Disable** Finally, uncheck the **Enable SDL** box and click OK to close & save settings. Re-open the Game Controllers dialogue again and uncheck **Enable SDL** again, and then navigate to the XInput tab. Confirm that nothing happens when you press any of the controller buttons on the Steam Deck in the controller detected on User #0 and then click OK to close the dialogue. Close out of the Wine Control panel.

Once the controllers have been disabled for the Wine prefix, you are now ready to play Kanvas on your Steam Deck. You'll be prompted for your root password to run the Kanvas Linux script after pressing Play in Lutris, so be sure you enter that in or else the web server will not start. During gameplay, I find that much of the default Steam controller key bindings work fine for me to move around in ROBLOX, though you may want to make your own changes. All these settings can be found within the Steam app, and it should be mentioned that Steam must be running or else you will not be able to navigate around Desktop Mode using the controls.
