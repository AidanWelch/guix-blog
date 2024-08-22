# 000 - Building a (non)Guix System Install Image

*initially written on August 21st, 2024*

## What is Guix?

The typical line is something like "Guix is a package manager designed to enable
reproducible environments, with a focus on libre software."  But, what I value
Guix, and it's siblings Nix and Aux, for is as tools that enable me to manage
what lives on my computer in as much of a *deterministic* way as possible. And,
furthermore to centralize that in just a few files, written in human readable
(and Git diff-able) text, which in the case of Guix is the Guile implementation
of the Scheme language.  Imagine on your current computer if you could include
all of the programs you need, none of what you don't need, and not have to
remember all the random configuration steps you had to to do.

Guix System is a (GNU)Linux distribution that takes that to the extreme, by
making Guix the core package manager of your computer- with Guix managing as
much of the system configuration as possible.

Guix development is led by the GNU Project, a free(libre) software collaboration
project.  GNU is behind a lot of important projects you've used, or at least
used by the software you use.  GNU also has a strong stance on what software
should be used, and so the packages listed in the main Guix [channel](https://guix.gnu.org/manual/en/html_node/Channels.html)
must somewhat align with that stance.  Which in some respects, to some users,
can hamper practical usage of Guix.

## A Brief Preface on the Use of Non-Libre Software

I respect the GNU project's push for only including FOSS(free and open source)
software, and I appreciate the ecosystem of FOSS software GNU has created. But,
to be totally honest, I do not have the energy nor motivation fiddle with trying
to find freedom respecting [Wi-Fi chips](https://guix.gnu.org/manual/en/html_node/Hardware-Considerations.html)
or keyboards.  And, its especially problematic when it comes to NVIDIA graphics
drivers.  So, I will settle for a not totally FOSS OS- but [as Nonguix says](https://gitlab.com/nonguix/nonguix):

> Those packages are provided as a last resort, should none of the official Guix
packages work for you.

Whenever possible, I will use free(for the un-Gnu-initiated [free, in this case,
means FOSS essentially](https://www.gnu.org/philosophy/free-sw.html)) software-
but when not practical I will use non-free software.  I will try to mention
whenever I use it in this blog, so those motivated to avoid it will do so.  I
encourage you to [read what GNU says about it and decide for yourself.](https://www.gnu.org/philosophy/free-software-even-more-important.html)

If you decide to stick with GNU Guix, the rest of this article is useless to you,
instead you should follow the [official Guix installation instructions.](https://guix.gnu.org/manual/en/html_node/System-Installation.html)

## A (Slightly) Briefer Preface on the Purpose of this Blog

I'm just going through the steps of making Guix usable as a distro for me.  I
am writing about it to contribute to the existing body of guidance and examples
to try to make using Guix more accessible to others as well as as reference
for me to look back on.  A lot of what's included in here may just be repeating
steps covered in official Guix and Nonguix sources, I hope that what I write can
still be valuable to fill in any slight gaps that others may be lost or confused
about.  I will assume general knowledge of Linux, a Bash shell, and Git- but
will try to explain as much as possible regardless.

A formatting note, I will try to use "\$" before commands when listing them, to
show it is a command to run in a terminal(unless specified otherwise), the "\$"
is not itself part of the command.  For example: `$ example_command`

## Installing Guix(the Package Manager)

Before a Nonguix image can be built you must have the Guix package manager
accessible.  Most package managers should have Guix available, for Debian-based
package managers you can do `$ sudo apt install guix`. (Thought you may want to
run `$ sudo apt update` and `$ sudo apt upgrade` first.)

If you're on Windows this is a great use-case for [WSL.](https://learn.microsoft.com/en-us/windows/wsl/about)
But, a virtual machine or other method could also be used.

Once Guix is installed, try running `$ guix pull`.  On Debian-based distros its
very slow, if you're impatient it should be fine to cancel with `Control+c`.

If you get the error:
``guix pull: error: failed to connect to `/var/guix/daemon-socket/socket': No such file or directory``
You need to start the Guix-daemon yourself, likely with
`$ sudo systemctl start guix-daemon`(systemd) or
`$ sudo service guix-daemon start`.

For the latter, you may get the error:
`/etc/init.d/guix-daemon: line 35: daemonize: command not found`

In that case you must install the packages `daemonize` and `opensysusers`, in
Debian-based distros with `$ sudo apt install daemonize opensysusers`.  ([It is
possible to avoid opensysusers, but more of a hassle.](https://salsa.debian.org/debian/guix/-/blob/debian/latest/debian/guix.README.Debian?ref_type=heads))
You can then run `$ sudo service guix-daemon start` to start the daemon, and
`$ sudo update-rc.d guix-daemon defaults` to autostart the daemon(supposedly,
though I've not found this to actually work- for unknown reasons).

## Adding Nonguix as a Channel

Guix channels basically act as cookbooks- telling Guix how to make your packages.
Nonguix is the primary channel for non-free(but it's still \$0) software.  You
have to add Nonguix as a channel before you can build the Nonguix system image,
luckily that's as easy as editing `~/.config/guix/channels.scm`(such as with
`$ nano ~/.config/guix/channels.scm`) and adding the following text to the
end of the file:

```scm
(cons*
	(channel
		(name 'nonguix)
		(url "https://gitlab.com/nonguix/nonguix")
		;; Enable signature verification:
		(introduction
			(make-channel-introduction
				"897c1a470da759236cc11798f4e0a5f7d4d59fbc"
				(openpgp-fingerprint
					"2A39 3FFF 68F4 EF7A 3D29  12AF 6F51 20A0 22FB B2D5"
				)
			)
		)
	)
	%default-channels
)
```

Then run `$ guix pull` to get the channel; it will take a while to run but do
not cancel it.

## Cloning the Nonguix Repository

You need to clone the Nonguix repo so you can build the install image from it,
simply run `$ git clone https://gitlab.com/nonguix/nonguix.git`.  If you don't
have `git` installed: install it with your package manager, such as
`$ sudo apt install git`.

## Building the Image

Now simply run:

`$ guix system image --image-type=iso9660 ./nonguix/nongnu/system/install.scm`

Again, it will take a while to run but do not cancel it.

If you get an error that says something like:
``Throw to key `encoding-error' with args `("scm_to_stringn" "cannot convert wide string to output locale" 84 #f #f)'.``
Or
`guix system: error: corrupt input while restoring archive from #<closed: file 7fc0cb767d20>`.
You may need to change `/etc/default/locale`(such as by running
`$ sudo nano /etc/default/locale`) from `LANG=C.UTF-8` to something
like `LANG=en_US.UTF-8`(or if you prefer another locale).  Once that's done you
should restart for the locale change to take effect.  In WSL rather than
restarting in Linux you should run `$ wsl --shutdown` in a Command Prompt or
Powershell window then start WSL again.

In the unlikely event you want a writable image(if you don't know, you probably
don't):

`$ guix system image --image-size=7.2GiB ./nonguix/nongnu/system/install.scm`

(`image-size` should be however much you need)

## Writing the Built Image to a Removable Drive

Once your build finishes the last line printed is the path to your built image,
in my case `/gnu/store/qa1drrr1axhj1wk7x7q5z5ibj5a8c1qb-image.iso`.

Plug a USB stick into your computer.  [If you prefer to burn a DVD refer to the
official Guix manual.](https://guix.gnu.org/manual/en/html_node/USB-Stick-and-DVD-Installation.html#Burning-on-a-DVD)

On Linux, [identify the name of the USB stick.](https://wiki.archlinux.org/title/File_systems#Identify_existing_file_systems)
You can then write the built image to the USB drive with the following command,
filling in the relevant sections with your specific values:

`dd if=[Path To Built Image] of=/dev/[USB Drive Name] bs=4M status=progress oflag=sync`

For example, assuming the name of the USB stick is `sdX` and using my above
built image path:

`dd if=/gnu/store/qa1drrr1axhj1wk7x7q5z5ibj5a8c1qb-image.iso of=/dev/sdX bs=4M status=progress oflag=sync`

On Windows with WSL, it's probably easier to move the built image to your
Windows drive, then write it to a disk. (But if you really want to [follow these
steps, then you can use the Linux instructions above.](https://learn.microsoft.com/en-us/windows/wsl/connect-usb))

So first copy the built image to your C drive Downloads folder with(of course,
filling in the bracketed details):

`cp [Path To Built Image] /mnt/c/Users/"[Windows Username]"/Downloads/nonguix-image.iso`

For example, for my image, and Windows user:

`cp /gnu/store/qa1drrr1axhj1wk7x7q5z5ibj5a8c1qb-image.iso /mnt/c/Users/"Aidan"/Downloads/nonguix-image.iso`

Now, locate `nonguix-image.iso` in your `Downloads` folder, and use a tool such
as [Rufus](https://rufus.ie/en/) or [balenaEtcher](https://etcher.balena.io/) to
write the image to your USB drive.  In Rufus if you don't select `DD` mode when
given the option, it may crash.  Once you're done, its safe for you to delete
the image both from your `Downloads` folder.

## Next

If you're not concerned about losing previous Guix generations, you can delete
up the built images from your hard drive with `$ guix gc --delete-generations`.

In `001` I will cover what to do with this USB stick.  If you had any problems
with this process, [submit a question to github.com/AidanWelch/guix-blog/issues](https://github.com/AidanWelch/guix-blog/issues)