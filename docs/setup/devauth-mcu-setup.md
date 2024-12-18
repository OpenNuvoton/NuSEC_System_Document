# DevAuth for target device setup

The firmware enables a target device (slave MCU) to receive commands from
a programmer(host MCU).

### Key configuration

The data transfer between a target device and a programmer is encrypted with
the shared key generated via ECDH. To set up for ECDH key exchange,
please edit the values of following variables in `keys.h`.

* `RESP_PRIKEY`: Local (target device) private key
* `CMD_U0PUBKEY_X`: X coordinate value of peer's (programmer) public key (first 32-byte)
* `CMD_U0PUBKEY_Y`: Y coordinate value of peer's (programmer) public key (last 32-byte)

!!! tip
    In most cases, the user could use the same key values as set in `.env`
    file of `NuSEC.py`.
