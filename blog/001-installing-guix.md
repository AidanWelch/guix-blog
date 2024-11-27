# 001 - Installing Guix System

*initially written on November 11th, 2024*

## Launching the Guix Installer

If you followed the instructions of `000` you should now have a USB stick with
the Guix bootable installer burned to it.  Turn off the computer you wish to
install Guix to; plug in the USB stick.  Now turn the computer on, it should
tell you how to open a BIOS/UEFI/boot menu, hold that key, if your computer
launches like normal turn it off and try again- this time holding that key
immediately.

> ❗ If no key was shown you can try holding `F10`, `F12`, `Escape`, or `Delete`
> and if none of those worked than try looking online for information on how to
> enter the boot menu for your computer or motherboard model.

Once you're in the boot menu there should be an option to select either a "boot
media" or to set a "boot order".  If you can select a "boot media" then all you
need to do is select the option most similar to "USB Media", "Removable Media",
or "USB HDD".  If only setting a "boot order" is an option then you need to move
the options like those to the top of the list.  Once you have done that there
should be an option to "Save and Exit".

> ❗ If you have difficulty with this there should be information about how to
> select a boot media for your specific computer or motherboard model online.

## Installing through the Guix Installer

You should now be on a screen where you're prompted to select a "Locale
Language", you can navigate this and future screens with the arrow keys on your
keyboard.  You can also type the first letter of the language of your choice.
I went with "English" and then press `Enter` to continue.  Next I select my
"territory for this language" in my case "United States", some languages might
not have this option.

You're now prompted to select how to install Guix, I will select "Graphical
install using a terminal based interface"- if you prefer to install manually I
recommend [this guide by System Crafters.](https://wiki.systemcrafters.net/guix/nonguix-installation-guide/)

I immediately get a red screen with the warning "Devices not supported by free
software were found on your computer", that's okay, I press `Enter`.  Next, I
select my timezone, you should know the drill by now: navigate with arrow keys,
submit with `Enter`.  Same with keyboard layout.

Next I'm prompted to choose a hostname, it doesn't really matter much what you
pick- but it is the name your computer will interact with networks with.  I
chose "guixpad", and again submitted with `Enter`.  Then I select "WiFi" to
provide internet access, I'm told "No wifi detected" but despite the earlier
warning if I press the `Right Arrow` key to go to "Scan" and press enter my
WiFi network shows up and I connect to it.  I'm then asked if I wish to enable
"Substitute server discovery" you're free to read it yourself, but I choose to
"Disable" it as I don't have a substitute server in my local network.

You should then be prompted to add an root/administrator password.  Next adding
a user so I press `Enter` and add the user "aidan".  I chose not to change the
"Real name" or "Home directory", you can again select a password- this time for
your user.  I press `Enter` on "OK" and continue to selecting a "Desktop
enviroment", I chose "GNOME" and then pressed the `Right Arrow` button so the
cursor moved to "OK" and pressed `Enter`.  I chose not to enable any of the next
options, and again navigated to "OK" and pressed `Enter`.  Again the same for
"CUPS".

When you have reached the `Partitioning method` screen either select
"Guided"/"Guided ... with encryption" or "Manual" and read their respective
sections below.

## Guided Partitioning

You should be given a list of hard drives you can install Guix on, press `Enter`
on the one you wish to install Guix to.

> ❗ **THIS PROCESS WILL DELETE ALL FILES ON THAT DRIVE** ❗

If asked you probably want a "gpt" partition table.

You are then given the option of seperating the "/home" partition, I chose not
to and to instead continue with "Everything is one partition".  I am shown the
partition layout and press `Right Arrow` to navigate to "OK" and press `Enter`.

> 🔐 Encrypted partitioning then prompts you to enter a password, make sure you
> remember this password.  Forgetting it likely means that all your files would
> be irrecoverable.  I then agree to format the new partition.

## Manual Partitioning

[TODO]

## Editing the Configuration file

The Guix install generated a configuration file for us based on the options we
chose, unfortunately this config doesn't include nonguix.  Make the following
changes to the *Scheme* code:

> **`λ`** Scheme is a functional programming language, and the primary language
> used in configuring Guix.  Being functional means that most things are written
> in the form of a function, the same kind of function as `f(x) = x + 3` but
> usually much more complex in effect.  In Scheme parenthesis(`(` and `)`)
> denote *nesting* what can be called "lexical depth" or just *depth*.  Each
> *depth* begins with a function name, followed by the *arguments* for that
> function.  It’s important in Scheme to be mindful of how deeply nested you are
> within parentheses- afterall this is what determines what function you're
> passing *arguments* to.
>
> As touched on in `000`: anything following `;;` in a line of Scheme code is a
> *comment* and not executed as code.

> 🗒️ Lines to add will be green, to remove will be red, other lines should be
> unchanged, even if they're slightly different on your computer.  Just make
> sure any changes you make are to the same *depth* I'm referring to.

```diff
- (use-modules (gnu))
+ (use-modules (gnu)
+              (nongnu packages linux)
+              (nongnu system linux-initrd))
(use-service-modules cups desktop networking ssh xorg)

(operating-system
+ (kernel linux)
+ (initrd microcode-initrd)
+ (firmware (list linux-firmware))
  (locale "en_US.utf8")
  (timezone "Europe/Warsaw")
  (keyboard-layout (keyboard-layout "us"))
  (host-name "guixpad")

  ;; The list of user accounts ('root' is implicit).
  (users (cons* (user-account
                  (name "aidan")
                  (comment "Aidan")
                  (group "users")
                  (home-directory "/home/aidan")
                  (supplementary-groups '("wheel" "netdev" "audio" "video")))
                %base-user-accounts))

  ;; Below is the list of system services. To search for available
  ;; services, run 'guix system search KEYWORD' in a terminal.

  (services
   (append (list (service gnome-desktop-service-type)
                 (set-xorg-configuration
                  (xorg-configuration (keyboard-layout keyboard-layout))))

           ;; This is the default list of services we
           ;; are appending to.
           %desktop-services))
  (bootloader (bootloader-configuration
                (bootloader grub-bootloader)
                ( targets (list "/dev/sda"))
                (keyboard-layout keyboard-layout)))
  (swap-devices (list (swap-space
                        (target (uuid
                                 "8204b016-5101-4d32-9aba-f4a7adcb9560")))))

  ;; The list of file systems that get "mounted".  The unique
  ;; file system identifiers there ("UUIDs") can be obtained
  ;; by running 'blkid' in a terminal.
  (file-systems (cons* (file-system
                         (mount-point "/")
                         (device (uuid
                                  "0be89db0-5391-44aa-9251-a87adb3ec25b"
                                  'ext4))
                         (type "ext4")) %base-file-systems)))
```

Basically what we've done there is import nonguix and install non-free firmware
and a non-free Linux kernel into our system.

There are also other changes that you

## Installing Guix System

**DO NOT PRESS OK**

Instead press the `Control`, `ALT` and `F3` keys simulatenously

Press `Enter` to go to a terminal screen.

From there you can type:

`$ herd start cow-store /mnt`

And finally:

`$ guix system init /mnt/etc/config.scm /mnt`