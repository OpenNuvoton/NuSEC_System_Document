# Quick start

First, OEM has to provision and configure a programmer for mass production.
OEM user will:

* Prepare programmer
* Export package

Next, a programmer and a package will be shipped to CM. Before mass production,
CM has to import package to programmer.

* Import package

Now the programmer is ready, trigger the production procedure to start
provisioning and programming a target device.

* Device authentication provisioning
* Firmware provisioning

Finally, if the target device is successfully programmed. The
`firmware_bl3_version` property of target device in database will be updated.
(By default it is `1.0.0`).

### Prepare programmer

```sh
python NuSEC_prepare_nulink3.py <com-port>
```

### Export package

Set the value of following variable to the UID of programmer. Make sure
the programmer has been registered in previous step. Otherwise, it will fail
to export a package.

* `programmer_uid`

By default the value of `prod_cnt` is 100. That means only 100 target
devices can be provisioned and programmed. OEM could adjust the value to
meet their requirements.

Now the BL2 & BL3 firmware for target device will be uploaded to cloud server.
Set the file path of BL2 & BL3 firmware in the script.

* `fw_bl2_path`
* `fw_bl3_path`

```sh
python NuSEC_pkg_export.py
```

The package will be exported to `encrypted_pkg.hex` file.

### Import package

```sh
python NuSEC_pkg_import.py <com-port>
```

### Device authentication provisioning

Start the mass production procedure by running this command.

```sh
python NuSEC_trigger_MP_flow.py <com-port>
```

The programmer will try to connect to Wi-Fi network and establish a TLS
connection with cloud server. Then, the programmer will make target board
perform key generation and provision certificate to the target board.

### Firmware provisioning

After device provisioning is complete, firmware provisioning automatically start.
The programmer make the target device ready for secure boot and then proceed to
burn signed firmware images onto target device. Once it is done, the target
device boots to application firmware (BL3).
