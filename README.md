# Guide for signing the Out-Of-Tree Kernel Modules Manually

## Introduction

This document provides information on signing the out-of-tree kernel modules for the Linux kernel. It explains the importance of module signing, the necessary configurations, key generation, and the process of signing and verifying modules that are developed outside the official Linux kernel source tree. By following this guide, you will ensure that your custom or third-party kernel modules are properly signed and verified to enhance system security and integrity.

#### Kernel Modules Signing:

Kernel module signing helps ensure that only trusted and verified modules are loaded into the Linux kernel, protecting the system from potentially malicious or unauthorized modules. When a module is signed, the system can verify its integrity, ensuring that it has not been altered.

#### Configurations Enabled for Module Signing in Kernel:

For module signing to work, specific kernel configurations are enabled:

- **`CONFIG_MODULE_SIG`**: Enables the signing and verification of kernel modules.
- **`CONFIG_MODULE_SIG_KEY`**: Specifies the key used to sign kernel modules.
- **`CONFIG_SYSTEM_TRUSTED_KEYRING`**: Includes the system's trusted keyring for module verification.

With these configurations, the kernel allows both signed and unsigned modules, but it verifies signatures when they are present.

#### What is an Out-of-Tree Kernel Module?

An out-of-tree kernel module is a module developed and compiled separately from the official Linux kernel source tree. These modules are typically third-party drivers or custom kernel extensions and are not included in the mainline kernel.

#### Issues with Loading Unsigned Out-of-Tree Modules:

If an out-of-tree module is unsigned and the system is configured to verify module signatures, the kernel may allow it to load but **log warnings**. The kernel does not enforce strict signature verification unless specific configurations (like `CONFIG_MODULE_SIG_FORCE`) are enabled. However, on systems where module signing is preferred, unsigned modules could raise security concerns or trigger kernel warnings.

For Example, we can observe following warning in `dmesg` :

```bash
[255400.502024] hello: loading out-of-tree module taints kernel.
[255400.502117] hello: module verification failed: signature and/or required key missing - tainting kernel
```

## Build setup and configuration

To sign and verify out-of-tree modules on the target, the key requirement is ensuring that your kernel is configured to support module signing (`CONFIG_MODULE_SIG`).

- **Ensure the kernel is configured** with module signing support (`CONFIG_MODULE_SIG=y`).
    
    ```bash
    # cat /boot/config-$(uname -r) | grep -i CONFIG_MODULE_SIG=
    CONFIG_MODULE_SIG=y
    ```
    

## **Signing Keys required for module signing**

To sign out-of-tree modules, you need a private key and a public certificate. These keys will be used to sign the modules and verify them when loaded.

#### Key Files for Module Signing:

- keys/private_key.pem: The private key used to generate the signature for the module.
- keys/public_cert.der: The public certificate associated with the private key.

## Signing Out-Of-Tree Kernel Module

To manually sign a module, use the `scripts/sign-file` tool available in the Linux kernel source tree. The script requires 4 arguments:

1. The hash algorithm (`sha256`)
2. The private key filename (`keys/private_key.pem`):  private key used for signing.
3. The public key filename (`keys/public_cert.der`): certificate that will be used by the kernel for verification.
4. The compiled out-of-tree module to be signed

To sign the a module `hello.ko` with SHA-256 hash using `private_key.pem` and `public_cert.der`, you can use the following command:

```bash
$ chmod +x script/sign-file
$ ./script/sign-file sha256 keys/private_key.pem keys/public_cert.der hello.ko
```

This command utilizes the kernel's `sign-file` script to apply a digital signature to the module, ensuring its integrity and authenticity.

# Verification of signed modules

1. **Check if the module is signed properly:**

```bash
$ modinfo hello.ko
filename:       <folder>/module-sign/hello.ko
description:    test kernel module
author:         Richa
license:        GPL
depends:
name:           hello
vermagic:       5.10.223-cip51+ind1 SMP preempt mod_unload modversions aarch64
sig_id:         PKCS#7
signer:         Siemens Industrial OS (db)
sig_key:        70:B8:B3:48:94:BA:30:06:0D:58:47:AB:64:FB:06:EE:9C:9A:EB:20
sig_hashalgo:   sha256
signature:      68:E6:7C:68:8D:0D:06:12:7D:C4:F4:CE:B1:8F:39:27:A9:38:55:66:
                C1:24:4C:9B:74:6A:86:2B:BE:0A:C8:18:50:79:01:80:22:27:33:9F:
                67:A5:7E:27:B8:40:C1:DA:16:94:05:B6:35:96:AC:2F:B4:C3:37:91:
                F7:C6:9D:6F:D1:1F:4A:9F:BE:B8:6C:CA:8E:C1:19:07:B3:01:B8:5E:
                00:89:35:C4:59:CB:0C:DE:60:0F:C5:58:AF:9C:CE:2F:71:03:F0:8B:
                E8:3D:C9:D6:F5:03:47:28:50:31:07:79:15:5E:73:6D:2F:CB:1A:64:
                E4:FC:3A:CB:E3:5B:DB:E0:46:EE:BA:DF:C5:2F:FD:8F:60:91:85:23:
                0D:61:F8:54:27:0D:64:44:54:E1:33:24:BB:E4:5D:3B:71:ED:29:6A:
                AD:9C:9E:A9:3A:35:50:D6:6E:D3:30:2D:CA:FF:71:66:A4:40:AD:C2:
                64:EE:B9:19:5C:CB:5F:C8:E3:9C:59:DA:26:22:1F:B7:37:0A:5C:6E:
                21:CC:98:8D:5C:6C:CE:97:BE:B1:A3:1B:2D:9A:AC:16:12:1E:23:21:
                28:19:BC:5A:8D:55:A7:F9:56:6B:18:62:10:10:98:FA:C9:F5:68:0C:
                FA:25:A3:F2:3B:11:57:09:85:FF:4B:6C:45:24:E0:3F
```

This command displays information about the signed module, including its signature details. Check the output for a "signature" field to confirm that the module has been properly signed.

1. **Check Kernel Logs**:
If there is an issue with module verification, check the kernel logs using `dmesg` to understand any failure in signature verification:

```bash
dmesg | grep "module verification"
```
