# Hardware CTF 4

This challenge will have you find and exploit bugs in a IoT webcam. The webcan is a commercial-off-the-shelf device with cusotomised firmware to deliver the CTF-style challenges.

Register your name/handle with InfoSect CTF to track your progress and compare yourself to others [http://ctf.infosectcbr.com.au:8000](http://ctf.infosectcbr.com.au:8000).

Solve each level to get the flag. Once you have the flag, enter it into the CTF to get your points.

## Flag 1 - Physical Access

Gain physical access to the UART on the webcam and establish a serial console. To do this, use a screw driver to pry off the case, and then pull out the circuit board. Be careful of the little "white" plastic circles that cover the screw holes - use tweezers to remove them, don't push. You can identify some header pins that will have UART access to give you a shell.

Read this section to learn how to connect to UART or look at the previous hardware CTFs on UART [https://infosectcbr.github.io/InfoSect-Hardware-CTF-1/] Level 1.

To connect the webcam to USB serial, connect GND on the webcam to GND on the serial bridge. TXO on the webcam to RX on the serial bridge. And RXO on the webcam to TX on the serial bridge. The webcam header pinout is as follows:

```
|                 |   |  |
|                 |___|  |
|         ______         |
\________|      |OOO_____/
                 |||
                 |||
    RX0 ---------`||           
    TX0 ---------` |
    GND ----------`
```

Install minicom on a Linux VM:

```
sudo apt-get install minicom
```

Pass through the USB serial to your VM via removable devices in VMWare.

Run minicom:

```
sudo minicom -D /dev/ttyUSB0
```

Configure minicom by typing ctrl-a then z on its own. Use menu option O and select serial port setup. Disable software and hardware flow control using F. The baud rate is 115200. Exit out of the configuration and hit enter a couple of times to see your shell!

Sometimes in UART you are presented with a root shell without needing to login. Other times you will have to enter a password...

## Flag 2 - Root shell

Take a look at the provided `firmware.bin`. Use tools such as `binwalk`, `squashfs-tools` and `john` to extract the root filesystem, find the `shadow` file and crack a password for logging into the device.

Now you have a shell, look around the filesystem for an obvious file containing the flag.

## For fun - Blinking lights

If you have remote access to a device, you can completely control it. As an example, try the following examples:

```
echo 0 > /sys/class/gpio/gpio48/value
echo 0 > /sys/class/gpio/gpio39/value
echo 1 > /sys/class/gpio/gpio48/value
echo 1 > /sys/class/gpio/gpio39/value
```

## Flag 3 - Ports Incoming

On the device, run the command `netstat -a`. Look at the man page for netstat to see other options, including an option to show the process name associated with each listener.

You should see that lighttpd is listening on 2 ports. One of these ports has a webserver, the other is serving a flag.

## Flag 4 - HTTP Headers

You have been provided with some leaked source code for lighttpd. It makes reference to a backdoor that has been inserted into the custom web server. To trigger the backdoor, you need to request `/index.html` from the server and use the correct HTTP header after the GET request. What executables are present on the camera to access web pages? Netcat and telnet aren't present. What other command can you use?

## Flag 5 - Buffer Overflow

There is a buffer overflow in the lighttpd custom webserver when it processes another HTTP header. The overflow overwrites a variable that causes a flag to be sent back to the requestor. You can try to audit the source code to find the bug, overflow the bug on the device to get the flag. Alternatively, you can try "fuzzing" the web server by sending strings in the headers and seeing what the device does.

If using `wget` on device, `--header` may not be the right option for the header you want to send. What other option in wget is associated with creating a header? This is probably the one you want to use.

Try sending a long strong of "A"s in this header. The length of the string has to be almost exactly correct. It's going to be less than 300 characters in length.

## Flag 6 - Firmware signing bypass

Insert an sdcard, take a look at the boot logs - do you notice an interesting file name? Maybe try seeing if there are any references to FAT or FAT32 which is a common filesystem for removable media.

Use the binary `check_filename` (on the device) to verify the name and reveal the flag.

## Flag 7 - Backdoor

Can you find the backdoor that is trying to connect to a remote site on the internet and making a reverse connection? Perhaps try using tcpdump and see if you can see any beacons or connection attempts. Once you know the domain name of the malicious server, try looking in /bin /sbin and /lib to see if there are any references to this domain name. Alternatively, can you see any suspicious processes running that implement this backdoor?

## GPL Release
As part of the GPL Copyright requirements, the source code modifications made to this webcam are available on request.



