# DevAuth setup

The first step of NuSEC.py is to download firmware (DevAuth) to a programmer.
The firmware enables a programmer (slave MCU) to receive commands from a PC
(act as host MCU).

### Hardware connection

(TODO)

### Key configuration

As mentioned in the [NuSEC.py setup](nusec-py-setup.md#create-env-file), 
the data transfer between NuSEC.py and a programmer is encrypted with
the shared key generated via ECDH. To set up for ECDH key exchange,
please edit the values of following variables in `MCU_KEY.h`.

* `RESP_PRIKEY`: Local (programmer) private key
* `CMD_U0PUBKEY_X`: X coordinate value of peer's (NuSEC.py) public key (first 32-byte)
* `CMD_U0PUBKEY_Y`: Y coordinate value of peer's (NuSEC.py) public key (last 32-byte)
