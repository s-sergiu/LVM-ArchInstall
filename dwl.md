# Installing DWL (a fork of dwm for Wayland);

In this installation we are gonna be using wayland as our display server instead of xorg so unfortunately the original dwm doesn't work with wayland, that's why we need to install dwl(a fork of dwm for Wayland);

Clone the dwl repo:
```
git clone git@github.com:jpohly/dwl
```

Install the dependencies you will need to dwl to work:
```
sudo pacman -S make, pkg-config, gcc, wlroots, wayland-protocols, polkit
```

You can ```cd dwl``` into the cloned repository and type ```make``` to compile it;
- you can test it by running ```./dwl``` inside the console and see if it loads correctly (you should see a blank screen with nothing);
- you can exit dwl by pressing ```Alt-Shift-Q```;

Make sure you create a symlink to ```~/.local/bin``:
- ```ln -s <absolute path of dwm binary> /home/user/.local/bin```;

Install the terminal of your choice:
- ```sudo pacman -S alacritty```;
A menu of your choice (my personal preference is bemenu):
- ```sudo pacman -S bemenu```;
And a font:
You can use DejaVu fonts:
- ```sudo pacman -S ttf-dejavu```;

Or Monaco fonts (personal preference):
- ```git clone "https://aur.archlinux.org/ttf-monaco"```;
- ```pacman -S fakeroot```; (required by makepkg);
- ```makepkg -si```; (to install the cloned fonts);
