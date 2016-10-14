# lab4
Pi-to-Pi (and Pi-to-Laptop or Pi-to-Phone) Communication using Bluetooth Serial Port Profile (SPP)

As discussed in lab 3, serial communication can be used to connect a wide range of devices. As an example of what is posible, this lab will show how to use a serial over Bluetooth connection for RPi-to-RPi communication. After following these steps, you will be able to have one Raspberry Pi remotely connect to another Raspberry Pi.

There will be four main steps in this process: 
1. Editing the `dbus-org.bluez.servive` file to enable the serial port service
2. Trusting and Pairing the RPis using `bluetoothctl`
3. Setup host RPi using `rfcomm` with `getty`
4. Connect from the client RPi using `rfcomm` and `screen`

**Step 1: Editing the `dbus-org.bluez.servive` file to enable the serial port service (on both RPis)**
To communicate with serial over Bluetooth, we need to setup both RPis in Bluetooth compatibility mode and add the Serial Port Protocol as discussed [here on the raspberrypi.org forums](https://www.raspberrypi.org/forums/viewtopic.php?p=919420#p919420).

We will need to edit the `/etc/systemd/system/dbus-org.bluez.service` on both RPis. From a terminal: `sudo nano /etc/systemd/system/dbus-org.bluez.service`. Then add a -C to the end of the ExecStart line, and add an ExecStartPost line such that your file reads:
```
...
ExecStart=/usr/lib/bluetooth/bluetoothd -C
ExecStartPost=/usr/bin/sdptool add SP
...
```
Save the changes, and then reboot your RPi.

**Step 2:  Trusting and Pairing the RPis using `bluetoothctl` (on both RPis)**
The next step is to make the Bluetooth connections detectable using `hciconfig` to enable page and inquiry scans. 
```
sudo hciconfig hci0 piscan && hciconfig
```
Note the BD Address for each RPi. We will need to pair/trust the host address on the client and vice versa. For example, let's say the host address is `01:02:03:04:05:06` and the client address is `0A:0B:0C:0D:0E:0F` for the following example.

After doing this on both systems, we can use `sudo bluetoothctl` to launch the Bluetooth control tool. This will launch the bluetoothctl application. Notice that your bash command prompt has been replace by *[bluetooth]#*. Now, within this tool, we will need to scan, trust, and pair the devices using the following commands on the client (using the host's address):
```
scan on
trust 01:02:03:04:05:06
pair 01:02:03:04:05:06
exit
```

Repeat this process using `sudo bluetoothctl` on the host RPi (using the client's address):
```
scan on
trust 01:02:03:04:05:06
pair 01:02:03:04:05:06
exit
```

**Step 3: Setup host RPi using `rfcomm` with `getty`**
We are now ready to link our bluetooth serial port service to `getty` (that's short for "get tty"). On the host RPi, please enter the following command:
```
sudo rfcomm watch hci0 l getty rfcomm0 115200 vt100
```

**Step 4: Connect from the client RPi using `rfcomm` and `screen`**
Finally, we can connect and bind the Bluetooth to a serial port on the client RPI, and then launch a terminal with `screen`.  For example, if the host's address is `01:02:03:04:05:06`, then you would enter the following
```
sudo rfcomm bind 0 01:02:03:04:05:06
```
This will create a serial port `/dev/rfcomm0`, which we can then connect to via `screen` as follows:
```
sudo screen /dev/rfcomm0
```
If you get an error that says (screen: command not found), then please install screen with the following: `sudo apt-get install screen`.  Then repeat the command above to run screen.

To exit screen, press `CTRL+A` and then `K` (to kill the screen). You should see a yes/no question appear in the lower left corner, then press `Y` to confirm.

**Step 5: Release the RFCOMM serial ports (and reverse process)**
Now we can switch the process by reconfiguring the client as a host, and vice versa. To do this, we must first release the `rfcomm0` devices on **both** RPis with the following command:
```
sudo rfcomm release 0
```
Now you can repeat steps 3 and 4.

**CLEANUP**
When we are done, you probably want to untrust and remove the other RPi (for security, if nothing else).  From the commandline, use `sudo bluetoothctl`. Then use the `untrust` and `remove` commands together with the other RPi's address (replacing `01:02:03:04:05:06` in the following example). 
```
untrust 01:02:03:04:05:06
remove 01:02:03:04:05:06
```

# Optional fun: connecting via Bluetooth SPP to a Pi terminal from a Laptop (Mac or PC) or Android Phone (sorry no easy way on iOS)

Once you have setup the serial port profile service, you can repeat step 2 to trust and pair other Bluetooth devices such as laptop computers and even cell phones. Once paired, you can then setup the RPi as a host with step 3 above and connect via a terminal application. 

For example, on a Windows PC, you should first allow Bluetooth connections. Then pair with the RPi. Then right-click to the `Properties` option for the new Bluetooth device, and enable the `Serial port (SPP)` in the `Services tab`. Then launch `PuTTY` and select the `Serial` radio button under *Connection Types*. Click `Open` and you should see your RPi login screen.

Alternatively, on an Android phone, you can download a Bluetooth terminal program such as `Bluetooth Terminal` by `QWERTY`. Once connected, you should use the upper right options button to enable the "add new line" option. (Sorry there is no quick way to do this with an iPhone since iOS does not natively support the SPP Bluetooth service. However, there are several alternatives using Bluetooth Low Energy discussed on SO [here](http://stackoverflow.com/questions/17794469/is-serial-port-profile-spp-supported-on-ios-7-over-bluetooth-low-energy-v4-0) and [here](http://stackoverflow.com/a/30600034/6816646).)


