---
layout: post
title: Installing Stata on (Ubuntu) Linux
---

![](https://www.statalist.org/forums/filedata/fetch?id=1351289&d=1469795531&type=full)

Installing Stata properly on Linux is surprisingly cumbersome, and I've found that even [Statacorp's own guide](https://www.stata.com/support/faqs/unix/install-on-linux/) is incomplete and outdated. So here's mine.

<!--more-->

I have installed Stata in many Linux systems, but this tutorial focuses on Ubuntu 16.04 (specially in the last part). I will use Stata 14.2, but with minimal modifications it can be used for versions 13 through 15.
I'm also assuming you have a tarball (a `.tar.gz` file) with the appropriate installation files for your operating system, and that you have root privileges in your system.

# Installation

1. Change directory to wherever you have the `.tar.gz` file (e.g. the Downloads folder). There, create a temporary folder to store the installation files (e.g. `statainstall`), and extract the installation files to that folder.
```bash
cd ~/Downloads
mkdir statainstall
tar -xvzf Stata14Linux64.tar.gz -C statainstall
```

2. Create the installation directory, and change location into it.
```bash
sudo mkdir /usr/local/stata14
cd /usr/local/stata14
```

3. Run the install script.
```bash
sudo ~/Downloads/statainstall/install
```

### License

In order you configure the license file you just need to run `./stinit` (you'll need root privileges to write the file). Be sure to have the serial number, code and authorization. No need to disconnect from the interwebz here, even if you have an "alternative" license ;)

```bash
sudo ./stinit
```

### Add directory to path

In order to be able to launch Stata you'll need to add the installation directory to the system path.

1. Add one line at the end of your `.bashrc` file to add Stata to the unix path.
```bash
echo export PATH="/usr/local/stata14:$PATH" >> ~/.bashrc
```

2. You need to source your `.bashrc` so that the changes are effective:
```bash
source ~/.bashrc
```
(Or you can also restart your terminal)

After adding the install directory to yout path, you should be able to run the appropriate version of Stata from the terminal, e.g.:
```bash
xstata-mp
```
This launches the X-window (GUI based) version of Stata MP. You can also run Stata directly within the terminal with
```bash
stata-mp
```
Obviously you'll need to substitute `mp` for whatever version you have a license for.
At this point you can also delete the temporary installation folder:
```bash
rm -r ~/Downloads/statainstall
```

# Additional improvements

### Create Unity launcher

Even after successfully installing and running Stata, in Ubuntu it won't be available as an application in the dash, and it won't have a proper icon in the application launcher. We can easily fix this by creating a `.desktop` file for Stata.

```bash
sudo nano /usr/share/applications/stata14.desktop
```

In this newly-created, empty file just copy the following Desktop Entry, obviously adjusting it if you have a different version or flavor of Stata:

```
[Desktop Entry]
Version=14.2
Terminal=false
Icon=/usr/local/stata14/stata14.png
Type=Application
Categories=Education;Scientific;
Exec=/usr/local/stata14/xstata-mp -q
Name=Stata/MP 14
Comment=Perform statistical analyses using Stata.
StartupNotify=true
```

After saving this file you should be able to find Stata from the Unity dash, and when launched it should have its icon.

### Adding mimetype associations

1. Edit the `stata14.desktop` file we created in the previous section (I'm using `gedit` now, but you can use `nano` or whatever you prefer).
```
sudo gedit /usr/share/applications/stata14.desktop
```

2. Add the following parameters immediately after the last line of the `stata14.desktop` file:
```
MimeType=application/x-stata-dta;application/x-stata-do;application/x-stata-smcl;application/x-stata-stpr;application/x-stata-gph;application/x-stata-stsem;
Actions=doedit;use;view;graphuse;projmanag;semopen;
```

3. In the same file, add the following entries below `[Desktop Entry]` and all of its parameters:
```bash
[Desktop Action doedit]
Name=Start Stata and open do-file editor
Exec=/usr/local/stata14/xstata-mp -q doedit "%f"
[Desktop Action use]
Name=Start Stata and use file
Exec=/usr/local/stata14/xstata-mp -q use "%f"
[Desktop Action view]
Name=Start Stata and open viewer
Exec=/usr/local/stata14/xstata-mp -q view "%f"
[Desktop Action graphuse]
Name=Start Stata and open graph editor
Exec=/usr/local/stata14/xstata-mp -q graph use "%f"
[Desktop Action semopen]
Name=Start Stata and open structural equation model builder
Exec=/usr/local/stata14/xstata-mp -q sembuilder "%f"
[Desktop Action projmanag]
Name=Start Stata and open project manager
Exec=/usr/local/stata14/xstata-mp -q projmanag "%f"
```

4. [Coming soon]

# Known issues

- If you have a dark GTK+ theme enabled (e.g. the dark variant of [Arc](https://github.com/horst3180/arc-theme), my theme of choice), the X-window version of Stata looks awful. I tried solving this by executing `xstata` with a different GTK theme; i.e. something like
```bash
Exec=env GTK_THEME=Arc xstata-mp
```
but apparently GTK+ 3 applications have to be coded to respect the `GTK_THEME` env variable, and Stata is not. I suspect some answers in [this](https://askubuntu.com/questions/78088/can-i-apply-a-different-gtk3-theme-from-the-main-one-to-an-individual-applicatio) and [this](https://unix.stackexchange.com/q/14129) questions might contain possible solutions, but I just switched to a light variant of Arc and called it a day.