# KPIFOCUS: KeeP my Input FOCUS from being stolen by hostile applications

A **dirty and brute-force hack** into the *Gnome Desktop Environment* to prevent **most** applications from **stealing the input focus**.

## Why

### Story
1. Imagine that you have a *slow* laptop. Your tasks today are
	* upgrade system using apt
	* fix bugs in 2 projects
	* play Minecraft
2. Now your laggy system boots up. You clicked *Terminal*, *Visual Studio Code*, *Minecraft Launcher* one by one in the app launcher.
3. *Terminal* shows up first, you typed `sudo apt upgrade -y`, *sudo* asked you for authorization.
4. When you're typing your password, *Visual Studio Code* popped up and **took your focus**, now you're typing password into wrong place !
5. You switched manually to *Terminal* and typed password again. *apt* is working, you switched back to *Visual Studio Code* to fix bugs in your project.
6. And then *Minecraft Launcher* popped up and **took focus** when you're typing, just voiding the function prototype you've just typed. You clicked 'launch' button, game still took some time to launch, you went back to code.
7. Now *Minecraft* launched, the window **took focus** and cover the whole screen. You're so annoyed and smashed at the power button. system reboots.
8. everything above would *just replay* once your system boots up again.

### Complain
1. I believe that it is us human beings who use computers for work, game or whatever else they prefer, rather than computer programs to confuse or 'play' humans. In other words, human control computers, not computers control human. It is true at least in the year 2022 and the past.
2. I have no idea why applications tend to **steal input focus** from the user, and why *window system* of OS could allow so. It is **disruptive** and **show no respect** to users, I mean human beings, to interrupt them for trivial software events when they're concentrating on one specific application.
3. Many so called *famous* applications are actually *toxic and aggressive*, they steal focus on **every** tiny dialog they show. Below are some applications I've encountered. There must be *way way way* more.
	* Steam
	* Visual Studio Code
	* Java from OpenJDK (thus *most* java programs)
	* VirtualBox
	* Wine (aka wine-based Windows applications)
4. *Window system* is also to blame for **giving up focus too easily**.
5. There're many GUI toolkits/libraries who make **focus stealing** their default behavior, such as [*GLFW*](https://www.glfw.org/docs/latest/window_guide.html#GLFW_FOCUSED_hint). I suspect that *Qt*, *GTK* and many others also have this *insane default*. This makes *every* application based on them **steals focus** if the developer is not aware of focus and flip the switch explicitly.

### Objective
My desired behavior considering **input focus** is fairly simple.
1. Input focus should be controlled by the user. **NO** program except the *window system* itself could change input focus.
2. *Window system* should **respect the user** and change input focus **only as user instructed**.
3. As a workaround to existing applications, when a window has input focus, it can transfer focus among windows **belonging to the same process**.
	* Many applications rely on model dialogs to work properly, such as *save as* dialog or *color picker* panel of *GIMP*
	* Applications can mess with their own dialogs once focused, but not other applications' nor when they lose focus. Applications playing around its windows too much would only ruin itself in user's experience.

## How

### Prerequisites
Again, this is a **dirty and brute-force hack** into the *Gnome Desktop Environment* to **solve focus-stealing issues**. Use this solution **AT YOUR OWN RISK**. Prerequisites as follows.

* software
	* Ubuntu 20.04 (works on 18.04, but needs manual patching)
	* **Gnome-shell** desktop environment, **Xorg** as *display server*  (**Wayland** works, not fully tested).
	* Being able to install packages & download source code via **apt**
* knowledge & skills
	* **KNOW WHAT YOU ARE DOING**
	* Use experience in *Gnome Desktop Environment*, which is the default for Ubuntu 18.04/20.04
	* Being able to run commands from **Terminal**
	* Being able to repair your system from **TTY** in case your *window system* blow up.
	* Knowledge about *Gnome-shell*, *Mutter*, *Xorg* helps.

### common steps
1. Enable **source-code-repositories** for *apt*
	* launch *Software & Updates*, check the **Source code** checkbox, confirm changes.
	* Alternatively, edit `/etc/apt/sources.list`, **uncomment** every line starting with `deb-src`, then run `sudo apt update`

### Hack *mutter*
1. Download source code for **libmutter-6-0**

		cd $PLACE_YOU_WANT_TO_DOWNLOAD_THE_CODE_INTO

		# libmutter-2-0 for Ubuntu 18.04
		apt source libmutter-6-0

		# directory name starting with 'mutter-', version number may change
		cd mutter-3.36.9/

2. Install build dependencies

		sudo apt build-dep libmutter-6-0

3. Apply the hack

		# for Ubuntu 18.04, you may need to modify the code manually
		git apply mutter-hack-focal.diff

4. Compile hacked **libmutter**

		DEB_BUILD_OPTIONS=nocheck dpkg-buildpackage -uc -b

5. Install the hack

		cd ..

		# file staring with 'libmutter-6-0_' and ending with '.deb', version number may change
		sudo dpkg -i libmutter-6-0_3.36.9-0ubuntu0.20.04.2_amd64.deb

		# prevent libmutter-6-0 from upgrading
		sudo apt-mark hold libmutter-6-0

6. Reboot your system and check the effect.

### Hack *xserver*
1. Download source code for **xserver-xorg-core**

		cd $PLACE_YOU_WANT_TO_DOWNLOAD_THE_CODE_INTO

		apt source xserver-xorg-core

		# directory name starting with 'xorg-server-', version number may change
		cd xorg-server-1.20.13/

2. Install build dependencies

		sudo apt build-dep xserver-xorg-core

3. Apply the hack

		# for Ubuntu 18.04, you may need to modify the code manually
		git apply xserver-hack-focal.diff

4. change code if needed

	currently the whitelist for SetInputFocus/GrabDevice is hardedcoded and set to */usr/bin/gnome-shell* . If you're not on Ubuntu/Gnome, change this to your window manager binary in file *dix/kpifocus.c*

5. Compile hacked **xserver-xorg-core**

		dpkg-buildpackage -uc -b --no-sign

6. Install the hack

		cd ..

		# file staring with 'xserver-xorg-core_' and ending with '.deb', version number may change
		sudo dpkg -i xserver-xorg-core_1.20.13-1ubuntu1~20.04.9_amd64.deb

		# prevent xserver-xorg-core from upgrading
		sudo apt-mark hold xserver-xorg-core

7. Reboot your system and check the effect.

### Note
Normally you could just proceed up to this point and git rid of **focus stealing** in *most cases*. Below are old methods before *xserver* hack is implemented. They have some downsides and **should not be used** unless you have a reason. See below for more details.

### Hack *libx11*

Usually the *libmutter* hack is enough for *most* applications. the hack below for *libx11* mainly blocked **Java** from stealing focus.  
This one is known to be **relatively tricky and unstable**, and could possibly break several other applications. If in doubt, **do not** apply this hack.

1. Download source code for **libx11-6**

		cd $PLACE_YOU_WANT_TO_DOWNLOAD_THE_CODE_INTO

		apt source libx11-6

		# directory name starting with 'libx11-', version number may change
		cd libx11-1.6.9/

2. Install build dependencies

		sudo apt build-dep libx11-6

3. Apply the hack

		# for Ubuntu 18.04, you may need to modify the code manually
		git apply libx11-hack-focal.diff

4. Compile hacked **libx11**

		dpkg-buildpackage -uc -b

5. Install the hack

		cd ..

		# file staring with 'libx11-6_' and ending with '.deb', version number may change
		sudo dpkg -i libx11-6_1.6.9-2ubuntu1.2_amd64.deb

		# prevent libx11-6 from upgrading
		sudo apt-mark hold libx11-6

6. Do configuration
	* Hack code inside *libx11* would reject focus requests other than process whose name matches environment variable `X_WINDOW_MANAGER_HOST`
	* You should set this variable to the *window manager* you're using. By default *mutter* on Ubuntu 18.04/20.04.
	* Since *libmutter* is hosted in *gnome-shell*, you should set it to `/usr/bin/gnome-shell`

			echo 'export X_WINDOW_MANAGER_HOST=/usr/bin/gnome-shell' >> ~/.profile

7. Reboot your system and check the effect.

### Hack *libxcb*

The hack below for *libxcb* mainly blocked **VirtualBoxVM** from **stealing focus & lock the keyboard**. This one is known to be **tricky and overkill**, and could break *VirtualBox itself* and possibly some other applications. Since the hack code relies on the fact that *libmutter* does not depend on *libxcb* to move around focus, so once this condition no longer meets, this hack could effectively **lock up or crash the desktop environment** even after a *normal* package upgrade. If in doubt, **do not** apply this hack. If you do want to try this hack, be ready to recover from a crash at any time in the future.

1. Download source code for **libxcb1**

		cd $PLACE_YOU_WANT_TO_DOWNLOAD_THE_CODE_INTO

		apt source libxcb1

		# directory name starting with 'libxcb-', version number may change
		cd libxcb-1.14/

2. Install build dependencies

		sudo apt build-dep libxcb1

3. Apply the hack

		# for Ubuntu 18.04, you may need to modify the code manually
		git apply libxcb-hack-focal.diff

4. Compile hacked **libxcb1**

		dpkg-buildpackage -uc -b

5. Install the hack

		cd ..

		# file staring with 'libxcb1_' and ending with '.deb', version number may change
		sudo dpkg -i libxcb1_1.14-2_amd64.deb

		# prevent libxcb1 from upgrading
		sudo apt-mark hold libxcb1

6. Reboot your system and check the effect.

### uninstallation / restore to system defaults
```
sudo apt install --reinstall xserver-xorg-core libmutter-6-0 libx11-6 libxcb1
sudo apt-mark unhold xserver-xorg-core libmutter-6-0 libx11-6 libxcb1
```

### How to save yourself when window system blew up
1. press **CTRL+ALT+F4** to switch to **TTY4**
	* any **TTY** without graphical interface works as well, you can even *ssh* into the system remotely
2. Type your username & password as promoted to login.
3. Type the following command, *sudo* would request your authorization, type password when asked.

		sudo apt install -y --reinstall xserver-xorg-core libmutter-6-0 libx11-6 libxcb1
		sudo reboot

4. Wait for the system to reboot. We've restored to default system libraries, now your *window system* should start normally.

## More details

### Known issues & defects
1. You'll notice that after the hack, **most** application launches without visible window but a notification (if you have not banned that). For example, when you click on downloaded document file in browser, the *Document Viewer* launches in background, whose window is covered by your browser, and on top bar _"Document Viewer" is ready_ pops up. **This is intended** since the "bring to front" behavior is actually a **focus stealing**. The application launches normally, its window is just behind the currently-focused window. You can click on the *Dock icon* or *Alt-Tab* to switch to that window. You have to get used to it, or just uninstall this hack since this solution is not for you.
2. Applying *mutter* hack & *xserver* hack should be enough for *normal* use under *Xorg*. The newly-implemented *xserver* hack could replace *libx11* hack and (mostly) *libxcb* hack. So don't use *libx11* hack, and if you're not using *VirtualBox* and rely on its *input grabbing* functionality, don't touch the *libxcb* hack.
3. Since this hack **violates** *ICCCM/EWMH protocol*, some applications may failed to get focus or even crash. I haven't observed this issue on any *normal* (I mean *native*) application I've used, but there might be.
4. There're still some applications who could **steal focus** even with *libmutter* hack applied. Those *toxic and brute* programs **directly send focus message** to *Xorg server* via either `XSetInputFocus` from **libx11** or `xcb_set_input_focus` from **libxcb**.
	* **Java** from *OpenJDK*. It calls `XSetInputFocus` on **every** dialog it creates. So nearly **every** java application with a main window would **steal your focus forcefully**. You can apply the *libx11* hack to block this. However some *Java* dialogs **failed to receive inputs** after applying this hack.
	* **VirtualBox**. Actually it's **fine** for *VirtualBoxVM* to mess with *Xorg*. When you're working in a VM, you want your input go directly into that VM, so *VirtualBoxVM* grabbed keyboard & mouse for you. However it has gone too far. If you have *auto capture keyboard* setting enabled (which is the default), when VM window first show up or *guest OS* change resolution, *VirtualBoxVM* would first request focus via *Qt5 framework*, which calls `xcb_set_input_focus`, and then **without checking the actual focus window** install keyboard & mouse grabs using *xcb library* (There's actually focus checking logic controlled by a compile-time switch, but it's off in official release. See more details in [VirtualBox source code](https://www.virtualbox.org/wiki/Downloads#ose) `VirtualBox-6.1.36/src/VBox/Frontends/VirtualBox/src/runtime/UIKeyboardHandler.cpp +280` ). Below are behaviors of *VirtualBoxVM* under different hacks.
		* No hack applied: *VirtualBoxVM* would **forcefully change to workspace it's on & raise itself & steal focus** once launched or *guest OS* change resolution.
		* With *libmutter* hack applied: workspace & window stacking won't change, but **keyboard is grabbed** once launched or *guest OS* change resolution, which means your keyboard suddenly **stopped working** since it's grabbed into the VM, and you cannot tell why if the VM window isn't visible, you have to press the *Host key* to release the keyboard grab.
		* With both *libmutter* and *libxcb* hack applied: issues above won't appear. However you may have trouble dragging to move/resize *VirtualBoxVM* window, pressing *Host key* before dragging sometimes works.
	* **Wine**. I haven't inspected the source code of *wine*, but since it acts like a *Windows API translator*, there must be very crazy code struggling to meet Windows behavior. The result is, after applying the hack, some Windows applications **failed to receive inputs** or even **crash on focus change**, some others **still steal focus**. This also applies to *Win32-based games* launched using *Steam Proton*.

### Q&A
1. Is the hack **safe** ?
	* If I said *yes*, would you believe me? Go and check the code yourself. Everything involved here is **open source**.
2. I don't know **how to install**
	* This solution is **not intended for people lack of technical background**, you need to modify & recompile system core libraries by yourself. You must know certainly what you're doing in every step and be able to repair your system in case of GUI crash. Please do not touch your system if you cannot understand steps above. If you really **hate focus stealing** and want to rule it out anyway, please have a try in a *virtual machine* first. **Do not risk your system & personal data**.
3. When installing package, error occurs like: *trying to overwrite shared '/usr/share/doc/libxcb1/changelog.Debian.gz', which is different from other instances of package libxcb1:amd64*
	* It seems that the library you compiled yourself is slightly different from the official release. I don't know the actual reason, but since it's just a *changelog*, I simply *delete this file* and try install again. You can backup the file if you want to be safer.
4. Why not submit patch/feature-request to respective *development groups* ?
	* It's actually a **hack** than a patch. It violates *ICCCM/EWMH protocol*, mess up the base code and make libraries *judge* its users, just to introduce weird functionality beneficial to a few people but confuse most normal users if it could ever appear in settings menu.
	* Further more, **focus stealing** is not a simple problem that could be solved by *mutter* alone. Cooperation among *libmutter*, *libx11*, *libxcb*, *Xorg* and possibly other related projects is required. However it is *hard* to negotiate so many teams to work together on such *trivial* and *confusing* feature.
5. What about **Wayland** ?
	* Hacks above **works** on **Wayland**, but some applications still **steals focus**. Tested on *Ubuntu 22.04 LTS* .Needs further reading & hacking the *mutter* code.

### Short-term plans & other things
* For *Java / VirtualBox / Wine*, it makes more sense to modify & recompile them instead of hack into *libx11 / libxcb*. However, since I don't have time and/or ability to set up their compilation environment & read through their source code, I chose the *easier* way to hack the system library instead.
* I'm neither a professor in *window system* nor a *hacker*. I'm just solving my very specific problem using my experience & technical skills and share my solution *in the hope that it will be useful*.
* I am fully employed, and have *many* projects/plans as off-time interests. So I may have little time to push this project forward ~or I'm just lazy~
* English is **not** my native language. I'm sorry for any misspoken or improper word/sentence.
