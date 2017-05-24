# FIOS-G1100
All the current information on reverse engineering the FIOS-G1100 Quantum Gateway router

Most of the original info here is from the [binwalk issue thread](https://github.com/devttys0/binwalk/issues/256)

### Hardware information
The cpu is an ARMv7 `Cortina G4`

### Current method of getting a root console ([@jameshilliard](https://github.com/jameshilliard))
You have to enable ssh using tr-069 on the WAN side(there's a built in remote activate-able root ssh backdoor), I set up a local genieacs server to do that. Redirecting the router to a local acs server is a bit tricky though, I originally tried to mitm it but that's not possible since the router verifies the acs server ssl certificate.

You can however change the config file to disable ssl and point it at your own acs server, the config file is aes encrypted but I have some python scripts that can decrypt and re-encrypt the config file so that it can be edited(I had to get some help with reversing the encryption scheme from the assembly for that).

These are the config file encryption/decryption scripts I'm using:
- [gwdecrypt.py](https://gist.github.com/jameshilliard/7112235b62dd929d69d7980c979ae7c0)
- [gwencrypt.py](https://gist.github.com/jameshilliard/99191b2a2877220041dc8789fa07339a)

### Information on serial console
The debug console is disabled for the UART pins on the router board.

In the uBoot logs, the router seems to be opening a rw console on UART0. There are apparently multiple serial ports named; UART0, UART1, UART2, and UART3

### Rolling back your firmware ([@Brandonv101](https://github.com/Brandonv101))
I also found a few hidden firmware rollback and update links assuming that the router is using the 192.168.1.1 IP: http://192.168.1.1/#/advanced/fwupgrade & http://192.168.1.1/#/advanced/fwrestore

### Firmware images/dumps
Currently nobody has a NAND dump of the older firmware that could hold the decryption/encryption keys/methods. Please make a pull request with the dump if you do!

([@jameshilliard](https://github.com/jameshilliard)) The firmware images are both signed and encrypted with PGP, the [signing](https://pgp.mit.edu/pks/lookup?op=vindex&search=0xC32877552D7B4FA1) key is also different from the encryption key.

I managed to find, extract and decrypt the pgp decryption key on the router for one of the firmware images(bhr4_release_01.03.02.02-FTR_firmwareupgrade.bin.signed).

```
-----BEGIN PGP PRIVATE KEY BLOCK-----

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
=UtFV
-----END PGP PRIVATE KEY BLOCK-----
```

### Other potentially useful links etc
- http://snapon.lab.bufferbloat.net/~d/verizon_firmware/
- http://myplace.frontier.com/~firmware/