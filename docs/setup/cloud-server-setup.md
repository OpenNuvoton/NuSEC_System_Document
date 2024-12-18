# Cloud server setup

This article shows you how to setup a cloud server on __Linux__.
The distro used here is Ubuntu server 22.04.

## Create a certificate authority (CA)

The CA created here is backed by HSM. `softHSM` is used for demonstration.
In production environment a real HSM is recommended.<br>
Users can follow the instructions in the interactive notebook to create CA
step by step.

### Install prerequisite software

If you don't have a real HSM, please install `softHSM` first.
```sh
sudo apt install softhsm
sudo usermod -aG softhsm $USER
```

Reboot the machine for the changes to be applied:
```
sudo reboot
```

Python’s package manager, pip, is required to install following packages.
```sh
sudo apt install python3-pip
```

Before starting the installation, creating and activating
a virtual environment is recommended.
```sh
python3 -m venv .venv
source .venv/bin/activate
```

To open interactive notebook, install Jupyter Notebook:
```sh
pip3 install jupyter
```

Open `creating-a-ca.ipynb`:
```sh
jupyter notebook creating-a-ca.ipynb
```

As described at the begining of the notebook, install following packages:
```sh
sudo apt-get install opensc gnutls-bin libengine-pkcs11-openssl
```

### Follow the steps in the notebook to create a CA

!!! note
    Before executing commands, please make sure the placeholder wrapped with
    angle bracket(`<>`) is replaced with your own value.

Here is a summary to help you understand what the commands provided in the
notebook do:

* Initialize softHSM
* Generate a private key for root CA on softHSM
* Choose a directory to store all certificates and keep track of signed certificates
* Prepare a root CA configuration file for OpenSSL
* Use the root key to create a root certificate
* Generate a private key for intermediate CA on softHSM
* Prepare a intermediate CSR configuration file for OpenSSL
* Create a intermediate CSR
* Prepare a intermediate CA configuration file for OpenSSL
* Create a intermediate certificate by signing a intermediate CSR with the root key
* Prepare a client certificate configuration file for OpenSSL
* (Optional) Create a client certificate by signing a client CSR with the intermediate key
* Prepare a server certificate configuration file for OpenSSL

Now the intermediate CA is ready to sign a client or server certificate.
The client certificate can be used for client authentication. And the server
certificate enable servers to use HTTPS.

### Sign a client certificate

To run [NuSEC.py](nusec-py-setup.md), a client certificate is required.
Follow the steps to create a certificate for the client.

Create a private key.

```sh
openssl ecparam -name prime256v1 -genkey -noout -out client_key.pem
```

Generate a CSR using the private key

```sh
openssl req -new -sha256 -key client_key.pem -out client.csr
```

```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:TW
State or Province Name (full name) [Some-State]:Taiwan
Locality Name (eg, city) []:.
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Nuvoton
Organizational Unit Name (eg, section) []:Tool
Common Name (e.g. server FQDN or YOUR name) []:NuSEC.py
Email Address []:.
```

Use the intermediate CA to sign the CSR to create a certificate for client.
```
openssl ca -batch -config config/sign_client_csrs.ini -engine pkcs11 -keyform engine \
  -extensions client_cert -notext -create_serial \
  -in client.csr -out client_cert.pem
```

### Sign a server certificate

Users has to deploy a certificate to the HTTP API server mentioned below.
Follow the steps to create a certificate for the server.

Create a private key.

```sh
openssl ecparam -name prime256v1 -genkey -noout -out key.pem
```

Generate a CSR using the private key

```sh
openssl req -new -sha256 -key key.pem -out server.csr
```

```
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:TW
State or Province Name (full name) [Some-State]:Taiwan
Locality Name (eg, city) []:.
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Nuvoton
Organizational Unit Name (eg, section) []:Tool
Common Name (e.g. server FQDN or YOUR name) []:Nuvoton Certificate
Email Address []:.
```

Use the intermediate CA to sign the CSR to create a certificate for server.
```
openssl ca -batch -config config/sign_server_csrs.ini -engine pkcs11 -keyform engine \
  -extensions server_cert -days 375 -notext -create_serial \
  -in server.csr -out server.crt
```

### Create the certificate chain file

Concatenate the intermediate and root certificates to create the
CA certificate chain.

```sh
cat intermediate/certs/intermediate.crt certs/root.crt > intermediate/certs/ca-chain.crt
```

## Host a HTTP API server

NuSEC system provides a HTTP API server built with Node.js and Express.
The server exposes endpoints that a client (e.g. NuSEC.py & NuLink) use to
access the resources.

### Install Node.js

The first thing to do is to install Node.js using node version manager (nvm).
Please follow the latest directions [here][nvm]. Or, execute the following
commands which work as of the time of writing.
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install 20
```

Navigate to the project directory and install the dependencies:
```sh
cd nusec-server
npm install
```

### Install MongoDB Community Edition

Please follow the latest directions [here][mongodb]. Or, execute the following
commands which work as of the time of writing.

```sh
sudo apt-get install gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org
sudo systemctl enable mongod
sudo systemctl start mongod
```

#### (Optional) Allow other host to connect via a specific network interface

The user might want to connect to MongoDB instance from other host.
If so, please edit the configuration file of MongoDB.
Add IP addresses that MongoDB instance should listen for client connection.

```sh
sudo nano /etc/mongod.conf
```

```conf
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1, <new-ip-addr>
```

Restart the instance for the changes to take effect.

```sh
sudo systemctl restart mongod
```

### Setup softHSM

Please make sure softHSM has been installed on your machine. If not,
follow the instruction [here](#install-prerequisite-software).

Follow the directions below to initialize two tokens.
Keypairs for secure boot will be stored on these tokens.

```
softhsm2-util --init-token --free --label cmprogrammer --so-pin <so-pin-for-cmprogrammer-token> --pin <pin-for-cmprogrammer-token>
softhsm2-util --init-token --free --label mcu --so-pin <so-pin-for-mcu-token> --pin <pin-for-mcu-token>
```

### Create `.env` file

Navigate to the root of project directory and create a new file named `.env`.
Please add following environment variables to it.

!!! note
    Please make sure the placeholder wrapped with
    angle bracket(`<>`) is replaced with your own value.

```
DATABASE_URL = mongodb://127.0.0.1:27017
PROG_SOPIN = <so-pin-for-cmprogrammer-token>
PROG_PIN = <pin-for-cmprogrammer-token>
MCU_SOPIN = <so-pin-for-mcu-token>
MCU_PIN = <pin-for-mcu-token>
```

### Setup `sign_csr.sh` script

Please navigate to the root of project directory and edit `sign_csr.sh`.
Assign to `INI` variable with the path of `sign_client_csrs.ini`
(client certificate configuration file).
It should be located in the CA directory created in 
[this step](#follow-the-steps-in-the-notebook-to-create-a-ca).

```
INI=<path-of-ca-directory>/config/sign_client_csrs.ini
```

### Create directories for storing files

Files such as firmware or certificates will be stored in the directories
created by this script.
```sh
./create_dirs.sh
```

### Deploy the certificate

In [pervious section](#sign-a-server-certificate), a key and a server certificate
have been created.
Please make them available to node.js server 
(place these files in the root of project directory):

* `key.pem`
* `server.crt`
* `root.crt`
* `intermediate.crt`

### Install python packages

The server depends on python libraries to perform operations on softHSM.
In addition, imgtool (a python program) is used to sign firmwares.
Please navigate to the root of project directory and run:
```
pip3 install -r requirement.txt
```

### Run the HTTP API server

Now the server is ready. Navigate to the root of project directory and
run the server with following command:

```sh
npm start
```


[nvm]: https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions
[mongodb]: https://www.mongodb.com/docs/manual/tutorial/install-mongodb-on-ubuntu
