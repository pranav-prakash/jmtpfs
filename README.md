#Jmtpfs

##Description

A mtp over FUSE virtual filesystem ported to macOS. 

Allows you to use MTP (media transfer protocol) through libmtp to transfer files to android
devices that do not support UMS (anything > 4.4).

As mounting is accomplished through FUSE, the device should be accessible both from the finder and
any command line utility.

##Build & Installation

These are build directions for OSX

First install [osxfuse](https://osxfuse.github.io/)

Then get Homebrew if you don't already have it.

`brew install` the following:

`libmtp`
`libusb`
`libmagic`

Then after `cd`ing to the cloned directory run

`./configure`

Hopefully there are no errors.

Then run `make`

And finally

`make install`

This will build the jmtpfs binary and place it in /usr/local/bin.

##Usage:

The way Fuse works is that it builds a virtual file system given a mountpoint (we will use a folder for this purpose)

Let's mount our device to the desktop.

To do this, create a folder called nexus (or whatever you like) inside your desktop (or someother place if you prefer).
You only have to do this once.

Next run

`jmtpfs -o local [path to created folder]`

Where [path to created folder] is replaced by the posix path.

For example: `jmtpfs -o local ~/Desktop/nexus`

The `-o local` flag creates a local mount so that it appears in the finder sidebar.

You can then access the files on the device as if it were a normal disk.

Passing the `-l` option will list the attached MTP devices.

You can then choose which device to mount with the `-device` option.

You can pass `-h` for help.

Unmounting can be done via the finder (eject) or via the osx standard umount

If you want to get really fancy you can add a name and icon:

`jmtpfs -o volname="Nexus 5" -o volicon=/System/Library/CoreServices/CoreTypes.bundle/Contents/Resources/com.apple.iphone-4-black.icns ~/Desktop/nexus`


**Alternatively**, in the releases tab is a helper application that is essentially a wrapper application (created with Platypus) around the above shell script. Upon launch, it takes care of automatically creating a folder in the OSX-convetion /Volumes directory and mounts into that created directory. Upon unmount, it deletes that created folder and leaves /Volumes as it was before. It also includes the name and drive icon so the fuse mountpoint can easily be interacted with via Finder.


that will automatically run the above using a shell script upon launching the app. 

##Performance and implementation notes:

Libmtp (and I assume the MTP protocol itself) doesn't support seeking within a 
file or partial file reads or writes. You have to fetch or send the entire 
file. To simluate normal random access files, when a file is opened the entire
file contents are copied from the device to a temporary file. Reads and writes
then operate on the temporary file. When the file is closed (or if a flush or
fsync occurs) then if a write has occurred since the file was last opened the
entire contents of the temporary file are sent back to the device. This means
repeatedly opening a file, making a small change, and closing it again will
be very slow.

Renaming or moving a file is implemented by copying the file from the device, 
writing it back to the device under the new name, and then deleting the 
original file. This makes renames, especially for large files, slow. This
has special significance when using rsync to copy files to the device. Rsync
copies to a temporary file, and then when the copy is complete it renames the
temporary file to the real filename. So when rsyncing to a jmtpfs filessystem, 
for each file, the data gets copied to the device, read back, and then copied
to the device again. There is a true rename (but not move) supported by libmtp,
but this appears to confuse some Android apps, so I don't use it. Image files,
for example, will disappear from the Gallery if they're renamed.

##Credits

Credits to @kiorky and @JasonFerrara for their initial work.

## Other

If you're using this for the purpose of transferring files to android using a fuse file system, also look into adbfs which is slightly more stable
