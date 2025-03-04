---
title: Create test certificates - Azure IoT Edge | Microsoft Docs
description: Create test certificates and learn how to install them on an Azure IoT Edge device to prepare for production deployment. 
author: kgremban

ms.author: kgremban
ms.date: 01/03/2022
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
---

# Create demo certificates to test IoT Edge device features

[!INCLUDE [iot-edge-version-201806-or-202011](../../includes/iot-edge-version-201806-or-202011.md)]

IoT Edge devices require certificates for secure communication between the runtime, the modules, and any downstream devices.
If you don't have a certificate authority to create the required certificates, you can use demo certificates to try out IoT Edge features in your test environment.
This article describes the functionality of the certificate generation scripts that IoT Edge provides for testing.

These certificates expire in 30 days, and should not be used in any production scenario.

You can create certificates on any machine, and then copy them over to your IoT Edge device.
It's easier to use your primary machine to create the certificates rather than generating them on your IoT Edge device itself.
By using your primary machine, you can set up the scripts once and then use them to create certificates for multiple devices.

Follow these steps to create demo certificates for testing your IoT Edge scenario:

1. [Set up scripts](#set-up-scripts) for certificate generation on your device.
2. [Create the root CA certificate](#create-root-ca-certificate) that you use to sign all the other certificates for your scenario.
3. Generate the certificates you need for the scenario you want to test:
   * [Create IoT Edge device identity certificates](#create-iot-edge-device-identity-certificates) for provisioning devices with X.509 certificate authentication, either manually or with the IoT Hub Device Provisioning Service.
   * [Create IoT Edge CA certificates](#create-iot-edge-ca-certificates) for IoT Edge devices in gateway scenarios.
   * [Create downstream device certificates](#create-downstream-device-certificates) for authenticating downstream devices in a gateway scenario.

## Prerequisites

A development machine with Git installed.

## Set up scripts

The IoT Edge repository on GitHub includes certificate generation scripts that you can use to create demo certificates.
This section provides instructions for preparing the scripts to run on your computer, either on Windows or Linux.
If you're on a Linux machine, skip ahead to [Set up on Linux](#set-up-on-linux).

### Set up on Windows

To create demo certificates on a Windows device, you need to install OpenSSL and then clone the generation scripts and set them up to run locally in PowerShell.

#### Install OpenSSL

Install OpenSSL for Windows on the machine that you're using to generate the certificates.
If you already have OpenSSL installed on your Windows device, ensure that openssl.exe is available in your PATH environment variable.

There are several ways to install OpenSSL, including the following options:

* **Easier:** Download and install any [third-party OpenSSL binaries](https://wiki.openssl.org/index.php/Binaries), for example, from [OpenSSL on SourceForge](https://sourceforge.net/projects/openssl/). Add the full path to openssl.exe to your PATH environment variable.

* **Recommended:** Download the OpenSSL source code and build the binaries on your machine by yourself or via [vcpkg](https://github.com/Microsoft/vcpkg). The instructions listed below use vcpkg to download source code, compile, and install OpenSSL on your Windows machine with easy steps.

   1. Navigate to a directory where you want to install vcpkg. Follow the instructions to download and install [vcpkg](https://github.com/Microsoft/vcpkg).

   2. Once vcpkg is installed, run the following command from a PowerShell prompt to install the OpenSSL package for Windows x64. The installation typically takes about 5 minutes to complete.

      ```powershell
      .\vcpkg install openssl:x64-windows
      ```

   3. Add `<vcpkg path>\installed\x64-windows\tools\openssl` to your PATH environment variable so that the openssl.exe file is available for invocation.

#### Prepare scripts in PowerShell

The Azure IoT Edge git repository contains scripts that you can use to generate test certificates.
In this section, you clone the IoT Edge repo and execute the scripts.

1. Open a PowerShell window in administrator mode.

2. Clone the IoT Edge git repo, which contains scripts to generate demo certificates. Use the `git clone` command or [download the ZIP](https://github.com/Azure/iotedge/archive/master.zip).

   ```powershell
   git clone https://github.com/Azure/iotedge.git
   ```

3. Navigate to the directory in which you want to work. Throughout this article, we'll call this directory *\<WRKDIR>*. All certificates and keys will be created in this working directory.

4. Copy the configuration and script files from the cloned repo into your working directory.

   ```powershell
   copy <path>\iotedge\tools\CACertificates\*.cnf .
   copy <path>\iotedge\tools\CACertificates\ca-certs.ps1 .
   ```

   If you downloaded the repo as a ZIP, then the folder name is `iotedge-master` and the rest of the path is the same.

5. Enable PowerShell to run the scripts.

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
   ```

6. Bring the functions used by the scripts into PowerShell's global namespace.

   ```powershell
   . .\ca-certs.ps1
   ```

   The PowerShell window will display a warning that the certificates generated by this script are only for testing purposes, and should not be used in production scenarios.

7. Verify that OpenSSL has been installed correctly and make sure that there won't be name collisions with existing certificates. If there are problems, the script output should describe how to fix them on your system.

   ```powershell
   Test-CACertsPrerequisites
   ```

### Set up on Linux

To create demo certificates on a Linux device, you need to clone the generation scripts and set them up to run locally in bash.

1. Clone the IoT Edge git repo, which contains scripts to generate demo certificates.

   ```bash
   git clone https://github.com/Azure/iotedge.git
   ```

2. Navigate to the directory in which you want to work. We'll refer to this directory throughout the article as *\<WRKDIR>*. All certificate and key files will be created in this directory.
  
3. Copy the config and script files from the cloned IoT Edge repo into your working directory.

   ```bash
   cp <path>/iotedge/tools/CACertificates/*.cnf .
   cp <path>/iotedge/tools/CACertificates/certGen.sh .
   ```

<!--
4. Configure OpenSSL to generate certificates using the provided script. 

   ```bash
   chmod 700 certGen.sh 
   ```
-->

## Create root CA certificate

The root CA certificate is used to make all the other demo certificates for testing an IoT Edge scenario.
You can keep using the same root CA certificate to make demo certificates for multiple IoT Edge or downstream devices.

If you already have one root CA certificate in your working folder, don't create a new one.
The new root CA certificate will overwrite the old, and any downstream certificates made from the old one will stop working.
If you want multiple root CA certificates, be sure to manage them in separate folders.

Before proceeding with the steps in this section, follow the steps in the [Set up scripts](#set-up-scripts) section to prepare a working directory with the demo certificate generation scripts.

### Windows

1. Navigate to the working directory where you placed the certificate generation scripts.

1. Create the root CA certificate and have it sign one intermediate certificate. The certificates are all placed in your working directory. 

   ```powershell
   New-CACertsCertChain rsa
   ```

   This script command creates several certificate and key files, but when articles ask for the **root CA certificate**, use the following file:

   * `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`
   
   This certificate is required before you can create more certificates for your IoT Edge devices and leaf devices as described in the next sections.

### Linux

1. Navigate to the working directory where you placed the certificate generation scripts.

1. Create the root CA certificate and one intermediate certificate.

   ```bash
   ./certGen.sh create_root_and_intermediate
   ```

   This script command creates several certificate and key files, but when articles ask for the **root CA certificate**, use the following file:

   * `<WRKDIR>/certs/azure-iot-test-only.root.ca.cert.pem`  

## Create IoT Edge device identity certificates

Device identity certificates are used to provision IoT Edge devices if you choose to use X.509 certificate authentication. These certificates work whether you use manual provisioning or automatic provisioning through the Azure IoT Hub Device Provisioning Service (DPS).

Device identity certificates go in the **Provisioning** section of the config file on the IoT Edge device.

Before proceeding with the steps in this section, follow the steps in the [Set up scripts](#set-up-scripts) and [Create root CA certificate](#create-root-ca-certificate) sections.

### Windows

Create the IoT Edge device identity certificate and private key with the following command:

```powershell
New-CACertsEdgeDeviceIdentity "<name>"
```

The name that you pass in to this command will be the device ID for the IoT Edge device in IoT Hub.

The new device identity command creates several certificate and key files, including three that you'll use when creating an individual enrollment in DPS and installing the IoT Edge runtime:

* `<WRKDIR>\certs\iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>\certs\iot-edge-device-identity-<name>.cert.pem`
* `<WRKDIR>\private\iot-edge-device-identity-<name>.key.pem`

For individual enrollment of the IoT Edge device in the DPS, use `iot-edge-device-identity-<name>.cert.pem`. To register the IoT Edge device to IoT Hub, use the `iot-edge-device-identity-<name>-full-chain.cert.pem` and `iot-edge-device-identity-<name>.key.pem` certificates. For more information, see [Create and provision an IoT Edge device using X.509 certificates](how-to-provision-devices-at-scale-windows-x509.md).

### Linux

Create the IoT Edge device identity certificate and private key with the following command:

```bash
./certGen.sh create_edge_device_identity_certificate "<name>"
```

The name that you pass in to this command will be the device ID for the IoT Edge device in IoT Hub.

The script creates several certificate and key files, including three that you'll use when creating an individual enrollment in DPS and installing the IoT Edge runtime:

* `<WRKDIR>\certs\iot-edge-device-identity-<name>-full-chain.cert.pem`
* `<WRKDIR>/certs/iot-edge-device-identity-<name>.cert.pem`
* `<WRKDIR>/private/iot-edge-device-identity-<name>.key.pem`

## Create IoT Edge CA certificates

<!--1.1-->
:::moniker range="iotedge-2018-06"

Every IoT Edge device going to production needs a CA signing certificate that's referenced from the config file. This certificate is known as the **device CA certificate**. The device CA certificate is responsible for creating certificates for modules running on the device. It's also necessary for gateway scenarios, because the device CA certificate is how the IoT Edge device verifies its identity to downstream devices.

Device CA certificates go in the **Certificate** section of the config.yaml file on the IoT Edge device.

:::moniker-end

<!--1.2-->
:::moniker range=">=iotedge-2020-11"

Every IoT Edge device going to production needs a CA signing certificate that's referenced from the config file. This certificate is known as the **edge CA certificate**. The edge CA certificate is responsible for creating certificates for modules running on the device. It's also necessary for gateway scenarios, because the edge CA certificate is how the IoT Edge device verifies its identity to downstream devices.

Edge CA certificates go in the **Edge CA** section of the config.toml file on the IoT Edge device.

:::moniker-end

Before proceeding with the steps in this section, follow the steps in the [Set up scripts](#set-up-scripts) and [Create root CA certificate](#create-root-ca-certificate) sections.

### Windows

1. Navigate to the working directory that has the certificate generation scripts and root CA certificate.

2. Create the IoT Edge CA certificate and private key with the following command. Provide a name for the CA certificate.

   ```powershell
   New-CACertsEdgeDevice "<CA cert name>"
   ```

   This command creates several certificate and key files. The following certificate and key pair needs to be copied over to an IoT Edge device and referenced in the config file:

   * `<WRKDIR>\certs\iot-edge-device-ca-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>\private\iot-edge-device-ca-<CA cert name>.key.pem`

The name passed to the **New-CACertsEdgeDevice** command should not be the same as the hostname parameter in the config file, or the device's ID in IoT Hub.

### Linux

1. Navigate to the working directory that has the certificate generation scripts and root CA certificate.

2. Create the IoT Edge CA certificate and private key with the following command. Provide a name for the CA certificate.

   ```bash
   ./certGen.sh create_edge_device_ca_certificate "<CA cert name>"
   ```

   This script command creates several certificate and key files. The following certificate and key pair needs to be copied over to an IoT Edge device and referenced in the config file:

   * `<WRKDIR>/certs/iot-edge-device-ca-<CA cert name>-full-chain.cert.pem`
   * `<WRKDIR>/private/iot-edge-device-ca-<CA cert name>.key.pem`

The name passed to the **create_edge_device_ca_certificate** command should not be the same as the hostname parameter in the config file, or the device's ID in IoT Hub.

## Create downstream device certificates

If you're setting up a downstream IoT device for a gateway scenario and want to use X.509 authentication, you can generate demo certificates for the downstream device.
If you want to use symmetric key authentication, you don't need to create additional certificates for the downstream device.
There are two ways to authenticate an IoT device using X.509 certificates: using self-signed certs or using certificate authority (CA) signed certs.
For X.509 self-signed authentication, sometimes referred to as thumbprint authentication, you need to create new certificates to place on your IoT device.
These certificates have a thumbprint in them that you share with IoT Hub for authentication.
For X.509 certificate authority (CA) signed authentication, you need a root CA certificate registered in IoT Hub that you use to sign certificates for your IoT device.
Any device using a certificate that was issued by the root CA certificate or any of its intermediate certificates will be permitted to authenticate.

The certificate generation scripts can help you make demo certificates to test out either of these authentication scenarios.

Before proceeding with the steps in this section, follow the steps in the [Set up scripts](#set-up-scripts) and [Create root CA certificate](#create-root-ca-certificate) sections.

### Self-signed certificates

When you authenticate an IoT device with self-signed certificates, you need to create device certificates based on the root CA certificate for your solution.
Then, you retrieve a hexadecimal "fingerprint" from the certificates to provide to IoT Hub.
Your IoT device also needs a copy of its device certificates so that it can authenticate with IoT Hub.

#### Windows

1. Navigate to the working directory that has the certificate generation scripts and root CA certificate.

2. Create two certificates (primary and secondary) for the downstream device. An easy naming convention to use is to create the certificates with the name of the IoT device and then the primary or secondary label. For example:

   ```PowerShell
   New-CACertsDevice "<device name>-primary"
   New-CACertsDevice "<device name>-secondary"
   ```

   This script command creates several certificate and key files. The following certificate and key pairs needs to be copied over to the downstream IoT device and referenced in the applications that connect to IoT Hub:

   * `<WRKDIR>\certs\iot-device-<device name>-primary-full-chain.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary-full-chain.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-primary.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device name>-primary.cert.pfx`
   * `<WRKDIR>\certs\iot-device-<device name>-secondary.cert.pfx`
   * `<WRKDIR>\private\iot-device-<device name>-primary.key.pem`
   * `<WRKDIR>\private\iot-device-<device name>-secondary.key.pem`

3. Retrieve the SHA1 fingerprint (called a thumbprint in IoT Hub contexts) from each certificate. The fingerprint is a 40 hexadecimal character string. Use the following openssl command to view the certificate and find the fingerprint:

   ```PowerShell
   openssl x509 -in <WRKDIR>\certs\iot-device-<device name>-primary.cert.pem -text -fingerprint
   ```

   Run this command twice, once for the primary certificate and once for the secondary certificate. You provide fingerprints for both certificates when you register a new IoT device using self-signed X.509 certificates.

#### Linux

1. Navigate to the working directory that has the certificate generation scripts and root CA certificate.

2. Create two certificates (primary and secondary) for the downstream device. An easy naming convention to use is to create the certificates with the name of the IoT device and then the primary or secondary label. For example:

   ```bash
   ./certGen.sh create_device_certificate "<device name>-primary"
   ./certGen.sh create_device_certificate "<device name>-secondary"
   ```

   This script command creates several certificate and key files. The following certificate and key pairs needs to be copied over to the downstream IoT device and referenced in the applications that connect to IoT Hub:

   * `<WRKDIR>/certs/iot-device-<device name>-primary-full-chain.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary-full-chain.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-primary.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device name>-primary.cert.pfx`
   * `<WRKDIR>/certs/iot-device-<device name>-secondary.cert.pfx`
   * `<WRKDIR>/private/iot-device-<device name>-primary.key.pem`
   * `<WRKDIR>/private/iot-device-<device name>-secondary.key.pem`

3. Retrieve the SHA1 fingerprint (called a thumbprint in IoT Hub contexts) from each certificate. The fingerprint is a 40 hexadecimal character string. Use the following openssl command to view the certificate and find the fingerprint:

   ```bash
   openssl x509 -in <WRKDIR>/certs/iot-device-<device name>-primary.cert.pem -text -fingerprint | sed 's/[:]//g'
   ```

   You provide both the primary and secondary fingerprint when you register a new IoT device using self-signed X.509 certificates.

### CA-signed certificates

When you authenticate an IoT device with CA-signed certificates, you need to upload the root CA certificate for your solution to IoT Hub.
Then, you perform a verification to prove to IoT Hub that you own the root CA certificate.
Finally, you use the same root CA certificate to create device certificates to put on your IoT device so that it can authenticate with IoT Hub.

The certificates in this section are for the steps in the IoT Hub X.509 certificate tutorial series. See [Understanding Public Key Cryptography and X.509 Public Key Infrastructure](../iot-hub/tutorial-x509-introduction.md) for the introduction of this series.

#### Windows

1. Upload the root CA certificate file from your working directory, `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`, to your IoT hub.

2. Use the code provided in the Azure portal to verify that you own that root CA certificate.

   ```PowerShell
   New-CACertsVerificationCert "<verification code>"
   ```

3. Create a certificate chain for your downstream device. Use the same device ID that the device is registered with in IoT Hub.

   ```PowerShell
   New-CACertsDevice "<device id>"
   ```

   This script command creates several certificate and key files. The following certificate and key pairs needs to be copied over to the downstream IoT device and referenced in the applications that connect to IoT Hub:

   * `<WRKDIR>\certs\iot-device-<device id>.cert.pem`
   * `<WRKDIR>\certs\iot-device-<device id>.cert.pfx`
   * `<WRKDIR>\certs\iot-device-<device id>-full-chain.cert.pem`  
   * `<WRKDIR>\private\iot-device-<device id>.key.pem`

#### Linux

1. Upload the root CA certificate file from your working directory, `<WRKDIR>\certs\azure-iot-test-only.root.ca.cert.pem`, to your IoT hub.

2. Use the code provided in the Azure portal to verify that you own that root CA certificate.

   ```bash
   ./certGen.sh create_verification_certificate "<verification code>"
   ```

3. Create a certificate chain for your downstream device. Use the same device ID that the device is registered with in IoT Hub.

   ```bash
   ./certGen.sh create_device_certificate "<device id>"
   ```

   This script command creates several certificate and key files. The following certificate and key pairs needs to be copied over to the downstream IoT device and referenced in the applications that connect to IoT Hub:

   * `<WRKDIR>/certs/iot-device-<device id>.cert.pem`
   * `<WRKDIR>/certs/iot-device-<device id>.cert.pfx`
   * `<WRKDIR>/certs/iot-device-<device id>-full-chain.cert.pem`  
   * `<WRKDIR>/private/iot-device-<device id>.key.pem`
