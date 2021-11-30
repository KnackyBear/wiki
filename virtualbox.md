# Increasing the screen resolution of linux console with GRUB in VirtualBox

Supported resolutions can be found from the Grub command line:

  1. Press C when grub runs to enter the grub command line
  2. On the grub command line run the following commands:

  ```
    set pager=1    (To enable paging of long vbeinfo output)
    vbeinfo
    reboot         (When done)
  ```

When a nice resolution is found (I’ll use 1280×1024 below) take the following steps to change the resolution:

  1. Start your VM and log in.
  2. Open the file /etc/default/grub in your favourite editor.
  3. Add (or replace) the following lines:
```
    GRUB_GFXMODE=1280x1024 # width x height required - see below
    GRUB_CMDLINE_LINUX_DEFAULT="nomodeset"
    GRUB_GFXPAYLOAD_LINUX=keep
```
  4. Save the file and exit the editor
  5. Run the following command to update GRUB
  6. ``sudo update-grub``
  7. Restart your VM

Source: http://www.ronaldtoussaint.nl/2018/01/24/increasing-the-screen-resolution-of-linux-console-with-grub-in-virtualbox/