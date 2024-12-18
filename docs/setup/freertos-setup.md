# FreeRTOS setup

The firmware based on FreeRTOS enables a programmer for mass production.
To communicate with cloud server, NuSEC.py and target devices, the firmware
requires following configuration.

### Key configuration

As mentioned in the [NuSEC.py setup](nusec-py-setup.md#create-env-file), 
the data transfer between NuSEC.py and a programmer is encrypted with
the shared key generated via ECDH. Also, the data transfer between a programmer
and a target device is encrypted with the shared key.
To set up for ECDH key exchange, please edit the values of following variables
in `keys.h`.

* `RESP_PRIKEY`: Local (programmer) private key
* `CMD_U0PUBKEY_X`: X coordinate value of peer's (NuSEC.py) public key (first 32-byte)
* `CMD_U0PUBKEY_Y`: Y coordinate value of peer's (NuSEC.py) public key (last 32-byte)

!!! tip
    In most cases, the user could use the same key values as set in `MCU_KEY.h`
    file of `DevAuth`.

### Server configuration

Set the server hostname and port respectively in `server_host.h`.

* `SERVER_HOSTNAME`
* `SERVER_PORT`

### Wi-Fi configuration

Configure the following options to join Wi-Fi network.

* `clientcredentialWIFI_SSID`
* `clientcredentialWIFI_PASSWORD`

---

## FreeRTOS settings for prototype version of NuSEC

### Set client certificate & private key

Set the client certificate & private key generated at
[this step](cloud-server-setup.md#sign-a-client-certificate)
in `config_files/clientcredential_keys.h` file.

* `keyCLIENT_CERTIFICATE_PEM`
* `keyCLIENT_PRIVATE_KEY_PEM`

### Set CA root certificate

Setup the root certificate of CA to trust the server certificate.
In `nusec.h`, set `ROOT_CA_PEM`.
