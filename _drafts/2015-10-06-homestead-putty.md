---
published: false
---



## Using vagrant putty plugin with homestead
I'd prefer docker...

The putty plugin for vagrant
https://github.com/nickryand/vagrant-multi-putty


SET VAGRANT_DOTFILE_PATH=XXX && vagrant putty

path to your homestead .vagrant file (in your home dir)
XXX = ~/.homestead\.vagrant
**SET VAGRANT_DOTFILE_PATH=~/.homestead\.vagrant && vagrant putty**

create the putty command here
https://github.com/vitr/homestead-putty
