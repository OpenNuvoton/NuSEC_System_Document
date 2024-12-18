# NuSEC.py setup

This article shows you how to setup python environment for NuSEC.py
on __Windows__.

### Install prerequisite software

Make sure python 3 is installed on your machine. If not, download python
installer from the [official site](https://www.python.org/downloads/windows/).
After installation, please check that the python installation directory
is added to the PATH (system variable).

### Activate virtual environment

Create and activate an isolated python virtual environment to avoid
install packages globally.

```
python -m venv venv
.\venv\Scripts\activate.bat
```

### Install python packages required by NuSEC.py

Navigate to the root directory of NuSEC.py project and execute following
command to install packages from a requirements file.

```
pip install -r requirements.txt
```

### Prepare for mutual TLS handshake

To establish a mutual TLS connection with cloud server,
following files is required to run NuSEC.py:

* client_cert.pem
* client_key.pem
* ca-chain.crt

The key and certificate for client (NuSEC.py) is created in
[this step](cloud-server-setup.md#sign-a-client-certificate).
To authenticate the cloud server, a CA certificate chain file, which is created
in [this step](cloud-server-setup.md#create-the-certificate-chain-file),
is also required. The user should place these files in the root directory
of NuSEC.py project.

### Create `.env` file

NuSEC.py requires `.env` file to set up communication with cloud server
and a programmer. A user can refer to `.env.sample` file to create your own
`.env` file. Here is a brief explaination for each variable in the `.env` file.

* `HOSTNAME`: The hostname of the cloud server
* `PORT`: The port number to which the cloud server is listening
* `PRIV_KEY_CMD`: Local (NuSEC.py) private key for use in the ECDH exchange
* `PUB_KEY_RESP`: Peer's (programmer) public key for use in the ECDH exchange

!!! info
    The data transfer between NuSEC.py and a programmer is encrypted with
    the shared key (using AES algorithm). The shared key is computed using
    a key agreement algorithm called ECDH. To help you better understand
    ECDH, [here][ecdh-article] is a good article about ECDH scheme.
    `NuSEC_ecdh_example.py` is a sample code to generate two EC key pairs
    for ECDH key exchange.

!!! note
    Make sure the `HOSTNAME` here matches the ip address or the hostname 
    in the SAN(subjectAltName) field of cloud server certificate. Otherwise,
    the python script may complain that certificate is not valid for `HOSTNAME`.
    By default, the server certificate is issued for `www.example.com`.
    You could edit `hosts` file to map the ip address of cloud server to
    `www.example.com`. For example, if the ip address of your cloud server
    is `192.168.56.101`. You could add following line to `hosts` file of the
    system that runs NuSEC.py.
    ```
    192.168.56.101 www.example.com
    ```

[ecdh-article]: https://cryptobook.nakov.com/asymmetric-key-ciphers/ecdh-key-exchange
