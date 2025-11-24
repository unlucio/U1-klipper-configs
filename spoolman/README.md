# Spoolman for Multi tools

![Activate Advanced Mode](assets/biboard1.gif)
![Activate Advanced Mode](assets/biboard2.gif)

## Prerequisites

### This does not integrate with any rfid tags, it's just spooolman.

### A working Spoolman instance

Spoolman is just a webapp, you don't need to have it runing on your printer, and in this case you wil need to host it somewhere else.
You can find instruction on how to install standalone [Spoolman](https://github.com/Donkie/Spoolman) on its [wiki](https://github.com/Donkie/Spoolman/wiki/Filament-Usage-History).
You can choose 2 differtn patsh:

- [docker](https://github.com/Donkie/Spoolman/wiki/Installation#docker-install): the easiets in my opinion
- or you can go nuts with pyton's delirium versions and dependencies and choose a [standalone](https://github.com/Donkie/Spoolman/wiki/Installation#standalone-install) path

### Unlocking the U1's advanced mode

![Activate Advanced Mode](assets/advanced_mode.gif)

### Your printer's IP:

![printer's wifi ip](assets/ip.jpeg)

### Download the [multi tool config file for spolman](spoolman_multi_tool.cfg)

## What to do:

### Access your U1's [Fluidd](https://docs.fluidd.xyz/) interface

Stick your printer's ip in a browser
![access flluidd interface](assets/fluidd_access.png)

### Navigate to the config file page

![access flluidd interface](assets/configs.png)

## Conect to Spoolman:

### Find your `moonraker.conf`

![access flluidd interface](assets/moonraker.png)

and append the spoolman connection to the end of the file

```
[spoolman]
server: http://<spoolman ip>:<port>
```

![access flluidd interface](assets/moonraker2<<<>>>.png)

### Create the `custom` directory

![access flluidd interface](assets/create_custom_directory1.png)
![access flluidd interface](assets/create_custom_directory2.png)

### Upload the `spoolman_multi_tool.cfg` file in the newly created `custom` directory

![access flluidd interface](assets/upload_multitoo_conf1.png)
![access flluidd interface](assets/upload_multitoo_conf2.png)

### Create variable persistence file

Create a file called `variables.cfg` int he config root
![access flluidd interface](assets/variables1.png)
![access flluidd interface](assets/variables2.png)
![access flluidd interface](assets/variables3.png)

### Inlcude the new file in your `printer.cfg`

Copy the inlcude line form [multi tool config file for spolman](additions_to_printer.cfg)
![access flluidd interface](assets/include1.png)
![access flluidd interface](assets/include2.png)

### Restart Klipper

![access flluidd interface](assets/restart.png)

### Activate Spoolman Pannel

From Fluidd's main page, open the `3 dots menu`, and select `Adjust dashboard layout`
![access flluidd interface](assets/3dots.png)
![access flluidd interface](assets/layout.png)

find the spoolman pannel and verify it is active
![access flluidd interface](assets/spoolman.png)
!!! PAY ATTENTION !!!: I reorganized my fuidd's pannel as it please me, so your spoolman panel might not be in the same position as mine. Look for it.

## Done:

Congrats, you can now asign spoolman's spools to your tools
![access flluidd interface](assets/spoolman_select.png)

```

```
