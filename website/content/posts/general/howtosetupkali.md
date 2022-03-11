---
title: "How I setup Kali Linux"
date: 2022-03-10T18:57:04+01:00
---

_Every tinkerer is only as good as his tools._

This post emphasizes on how I setup my Kali Linux virtual machine, which will be used for all CTF challenges showcased on this website.

#### Step 1) Download

Download an official Kali Linux VM image from [Kali.org](https://www.kali.org/get-kali/#kali-virtual-machines).

Since I am using VirtualBox I am using the VirtualBox 64-bit image of Kali Linux. Make sure to pay attention to the text above the download that says _These images have the default credentials "**kali/kali**"_. Keep that in mind, because you will need the credentials later to log into the virtual machine.

#### Step 2) Import

Import the downloaded .ova file into VirtualBox.

Just open up VirtualBox, click on _Import_ and search for the .ova file on your computer. Click yourself through the import wizard, wait for the setup to finish and you are good to go.

#### Step 2.5) VM Configuration

This step is not necessary but can be useful if you want to have a say in how much resources you are giving to your Kali VM.

Once the import is done VirtualBox will present the newly generated VM in its list of local VMs. Right click on the Kali VM and select _Settings_. This will open the settings menu where you can change a lot of configurations for the VM.

Under _System -> Motherboard_ I like to change the Base Memory (which represents the amount of RAM you are giving to your VM) to 4096 MB. How much you can/should give depends on how much you have available on your actual system.

Under _System -> Processor_ you can change the amount of processors you are willing to give to the VM. Generally I leave it at the default 2, however if you feel like your actual PC struggles with that you can lower it to 1.

This is everything I do in terms of VM configuration though you can tweak a lot more than that.

#### Step 3) Login

Now you can start the VM from within VirtualBox and after a few seconds you should be presented with Kali's login screen.

Here is where we can input the default credentials _kali/kali_ given to us earlier to log into the machine.

Now this is where to real fun begins!

#### Step 4) Add new user and delete the default one

It is not really necessary but definitely good practice to replace the default user and it makes me feel more at home.

All of this can easily be done from the command line like this:

1. Create the new user: `sudo adduser <new_username>`
   1. Provide a password for the new user and leave all the other information empty.
   2. This will automatically generate a new directory under _/home/_.
2. Give the new user sudo privileges: `sudo usermod -a -G sudo <username>`
3. Log out of Kali and login again as the new user.
4. Delete the default user: `sudo userdel kali`.
   1. It is possible that you may have to restart the VM before you are able to do that.
5. You can now safely delete the _kali_ folder under _/home/kali_: `sudo rm -rd /home/kali`

#### Step 5) Change keyboard layout

Nothings worse than typing commands into the terminal while at the same time having to think in a different kezboard lazout.

Ok all jokes aside, since I am using a QWERTZ keyboard and not a QWERTY, which is the default for Kali, the first thing I do after changing user accounts is to go into the keyboard settings by opening the start menu, typing in _keyboard_ and selecting a QWERTZ layout in my language from the _Layout_ tab.

The only reason I did not do this before is because changing the user also resets your keyboard preferences.

#### Step 6) Configure aliases

Command line aliases are great for setting up command shortcuts of regularly used commands you would otherwise have to type in entirely. Just make sure you do not forget what is behind the alias.

This is the content of my `.aliases` file in `~/.aliases`

```bash
alias ls="ls --color=always -arlht"
function apt-updater {
   apt update &&
   apt dist-upgrade -Vy &&
   apt autoremove -y &&
   apt autoclean &&
   apt clean
}
```

The `ls` alias is to make calls to `ls` print out more detailed information on the directory I am currently in. Especially helpful when you want to see ALL files inside a directory and the color-encoding also helps to discern directories from files.

The `apt-updater` function is a quick and handy shortcut for keeping your Kali VM up to date. Just call it once a week and you can be sure that all your packages and applications are running on their latest version.

You could obviously create a lot more shortcuts, for example some `nmap` aliases that cut down lengthy `nmap` commands, but if you are just starting out with CTF challenges and Penetration Testing I would advice against it. You should first become familiar with the commands themselves before setting up aliases that abstract important handles and configurations away.

After you are done with the `.aliases` file we need to make sure that the aliases are loaded on shell start. For this add the following code to the end of the `.zshrc` (or `.bashrc` if you prefer bash) file in your user home directory:

```bash
if [ -f ~/.aliases ]; then
    . ~/.aliases
fi
```

Opening a new shell should now load in configured aliases automatically.

#### Step 7) Update repositories

After setting up the alias for updating your Kali VM I would advice you to do that now. Just call `apt-updater` from the command line and wait until `apt` is done with updating your machine.

_This step can take **A LOT** of time depending on your internet connection and the resources you gave to the VM. Go grab a tea or coffee and take a break._

#### Step 8) Install a Terminal Multiplexer

Now that your repositories are up to date you can get started with installing new tools.

Kali's default terminal emulator does its job but is fairly basic in terms of features. Therefore I like to use a terminal emulator that gives you more configuration options, which is why I like to install `terminator` and tweak it to my liking, though there are many more if you prefer to find one yourself.

To install and configure `terminator` like I do follow these steps:

1. Install `terminator` via `apt`: `sudo apt install terminator`
2. Go into the start menu and type in `Default Applications`
3. Under the `Utilities` tab you can find the entry for `Terminal Emulator`. Here you can select `Terminator` from the list and close the window again.
4. Opening a new terminal should now open a `Terminator`-styled terminal.
5. Right-click on the window and select `Preferences` from the context menu to open `Terminator's` configuration menu.
6. Under `Profiles`
   1. `General`: disable `Show titlebar`
   2. `Scrolling`: enable `Infinite Scrollback`
7. Under `Keybindings`
   1. Set `close_term` to `Ctrl+Super+X`
   2. Set `new_terminator` to `Ctrl+Alt+T`
   3. Set `split_horiz` to `Ctrl+Super+Down`
   4. Set `split_vert` to `Ctrl+Super+Right`

Feel free to tweak more options to your liking until you feel comfortable with the configuration. Also it is going to take some time remembering all the keybindings but I can assure you that it is worth it!

#### Step 9) Install a Code Editor

Kali comes pre-packaged with a fairly basic text editor and vim. However I like to work and code in Visual Studio Code, therefore I always add and install VS Code when setting up a new VM.

To follow along just type these commands into a terminal:

```bash
sudo apt-get install wget gpg
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
rm -f packages.microsoft.gpg
```

Then update the package cache and install VS Code like this:

```bash
sudo apt install apt-transport-https
sudo apt update
sudo apt install code
```

#### Step 10) Change SSH Keys

Last but not least a little security best practice.

Kali comes with default SSH keys that could easily be used for intercepting communication when you are controlling something else over SSH, therefore it is recommended to change your SSH keys after setting up a new VM.

Just type in the following commands and you are done:

```bash
cd /etc/ssh/
sudo rm -v /etc/ssh/ssh_host_*
sudo dpkg-reconfigure openssh-server
sudo systemctl restart ssh
```

#### Conclusion

Congratulations! You just set up and configured your very own Kali virtual machine. You took the first step in becoming a real Cyber Security professional and you are good to go for the challenges that await you in the world of CTF and Penetration Testing. Once you familiarized yourself with this new environment try to change settings and configuration to your liking; make it your own, unique hacking machine.
