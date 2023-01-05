# VT100

_Gilbert François Duivesteijn_



## Abstract

This document describes how to setup a self-made [VT100](https://www.tindie.com/products/petrohi/geoffs-vt100-terminal-kit/).



## On a Raspberry Pi

Connect the VT100 to the Raspberry Pi as follows:

| Raspberry Pi       | VT100    |
| ------------------ | -------- |
| `GPIO 14 GND`      | `J5 GND` |
| `GPIO 10 UART0 RX`      | `J5 TX`  |
| `GPIO  8 UART0 TX`     | `J5 RX`  |
| `GPIO  2 5V [optional]` | `J5 VCC` |
| `GPIO  4 5V`            | `J6 5V`  |
| `GPIO  6 GND`           | `J6 GND` |

Then type `sudo raspi-config`, to to `Interface Options` and enable `Serial port`.

By now, you should be able to connect and log in.

## On Ubuntu systems

For a single session, start
```sh
sudo /sbin/agetty 115200 /dev/ttyUSB0
```
This will give a login prompt at the console. The agetty process will end as soon as you log out.

### Setting the right $TERM environment

To make things work 100%, consider the following additional steps:

- In `/lib/systemd/system/serial-getty@.service`, change the `ExecStart` option to:

  ```
  ExecStart=-/sbin/agetty -o '-p -- \\u' 115200 %I vt100
  ```

  where 115200 is the default baudrate set in the VT100 and the Raspberry Pi.

- If you happen to have a VGA monitor that displays 80x30 instead of 80x24 characters, add the following line to your `~/.profile` :

  ```
  /usr/bin/stty -F /dev/ttyS0 rows 30
  ```

  You can check all stty settings with `stty -a`.



### Changing baudrate

For the full, original VT100 experience, set the baud rate to 9600 by following the steps below:

- In the VT100 setup screen, go to `Serial -> Baud rate ` and set it to 9600.

- On the Raspberry Pi, edit the file `/boot/cmdline.txt` and change the baud rate:

  ```
  console=serial0,9600 console=tty1 ...
  ```

  
