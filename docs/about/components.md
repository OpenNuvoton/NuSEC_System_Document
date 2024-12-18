# Components

NuSEC system is composed of software and hardware components. This page
introduces the components used by original equipment manufacturer (OEM) and
contract manufacturer (CM).

## Original equipment manufacturer (OEM)

### Cloud server

A cloud server enables OEM to controll the manufacturing process.
The server is equipped with a hardware security module (HSM) containing
secure boot keys which establish the root of trust on the target device.
OEMs' firmware is signed with private key stored in HSM and securely
transfer from cloud server to programmers over mutual TLS. Moreover,
device-specific certificates are generated and managed by cloud server.

### NuSEC.py

NuSEC.py is a set of Python tools to set up programmers for mass production.
It can deliver certificates to programmer for cloud onboarding. NuSEC.py can
help OEM to create production package, which contains production limit and
token for downloading signed firmware. Additionally, the package is encrypted
before being sending to contract manufacturer without exposing itself.

### Programmer (NuLink3)

NuLink3 is equipped with KeyStore, which is used to securely store signing
and encryption keys in hardware. Besides, NuLink3 is ready for secure boot.
These features enable NuLink3 to protect OEM's secrets. NuLink3 will be shipped
to CM after OEM finish the setup process.

## Contract manufacturer (CM)

### NuSEC.py

NuSEC.py is simply used to import production packages provided by OEM.
For now, CM can use NuSEC.py to trigger the mass production flow.

### Programmer (NuLink3)

A programmer is a host MCU, which is responsible for securely provisioning
and programming slave MCUs (target devices) under an unsecure manufacturing
environment. 

### Target device (Slave MCU)

A target device can generate private key inside the device and never leaves the
device. And each device is provisioned with the certificate signed with cloud
server. In addition, a target device supports secure boot, which means only
firmware signed by cloud server can be executed on the target device. It
prevents malicious code from compromising the boot process.
