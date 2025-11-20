<!-- omit in toc -->
# MAXAR_encryption_keys
<!-- omit in toc -->
## Contributors

* Björn Blissing, [@bjornblissing](https://github.com/bjornblissing)
* Johan Borg, [@jo-borg](https://github.com/jo-borg)

<!-- omit in toc -->
## Status

**Version 1.0.0**, June 25, 2025

<!-- omit in toc -->
## Dependencies

Written against the 3D Tiles 1.1 spec.


## Contents

- [Overview](#overview)
- [Optional vs Required](#optional-vs-required)
- [Schema](#schema)
- [Usage](#usage)
- [Examples](#examples)

## Overview

The `MAXAR_encryption_keys` extension describes the keys used to encrypt tile content. It is defined at the tileset level and provides a reference to an external keyfile containing encryption details.

This extension works in conjunction with the `MAXAR_buffer_encryption` glTF extension to enable secure delivery of 3D Tiles content.

## Optional vs Required

This extension is required; it must be listed in the tileset JSON `extensionsUsed` and `extensionsRequired` arrays.

## Schema

The extension uses the following schema file:

* [tileset.MAXAR_encryption_keys.schema.json](schema/tileset.MAXAR_encryption_keys.schema.json) - Schema for the tileset extension

## Usage

To use this extension:

1. Add the extension to both `extensionsUsed` and `extensionsRequired` arrays in the tileset JSON
2. Add the extension object to the tileset with a reference to an external keyfile
3. Use the `MAXAR_buffer_encryption` glTF extension in tile content to encrypt buffers

The keyfile must be referenced using a standard URI or IRI. Data URIs are not allowed for security reasons.

Relative URIs are resolved according to RFC 3986. When the keyfile URI is relative, it is resolved relative to the tileset JSON file.

The keyfile format and detailed field descriptions are documented in the `MAXAR_buffer_encryption` glTF extension.

**Note:** Key identifiers must be between 16 and 256 characters and uniquely identify the key within the system. The key ID should be generated to guarantee uniqueness (for example, using a UUID, a random string, a systematically generated id with namespace context, or a cryptographic hash of the key material). It must never contain the key itself or any reversible encoding of the key.

## Key Management and Security Model

### User Key vs Secret Key

The `MAXAR_encryption_keys` extension uses a two-tier key management system:

#### Secret Key (Data Encryption Key)
- **Purpose**: The actual key used to encrypt/decrypt the tile content data
- **Storage**: Encrypted and stored in the `encryptionKey` field of the keyfile
- **Location**: Part of the dataset, distributed with the 3D Tiles content
- **Security**: Protected by encryption using the user key

#### User Key (Key Encryption Key)
- **Purpose**: Used to encrypt/decrypt the secret key stored in `encryptionKey`
- **Storage**: **NOT stored anywhere in the dataset or keyfile**
- **Location**: Managed entirely by the viewing application
- **Security**: Responsibility of the application developer and end user

### Security Responsibilities

#### Dataset Provider Responsibilities:
1. Generate secure secret keys for data encryption
2. Encrypt tile content using the secret keys
3. Encrypt the secret keys using user-provided keys
4. Store encrypted secret keys in the keyfile `encryptionKey` field
5. Distribute the dataset with encrypted content and keyfiles

#### Viewing Application Responsibilities:
1. **Secure user key management**: Obtain, store, and protect user keys
2. **Key derivation**: Derive or obtain user keys through secure channels
3. **Secret key decryption**: Use user keys to decrypt `encryptionKey` values
4. **Memory protection**: Securely handle decrypted keys in memory
5. **Key lifecycle**: Manage user key rotation and expiration

#### End User Responsibilities:
1. Protect user keys from unauthorized access
2. Follow organizational security policies for key management
3. Ensure secure transmission of user keys to viewing applications

### Security Benefits

This two-tier approach provides several security advantages:

1. **Dataset Distribution**: The dataset can be distributed through insecure channels since it does not contain the user keys needed for decryption
2. **Key Separation**: User keys are managed separately from the data, reducing exposure risk
3. **Access Control**: Only authorized users with valid user keys can decrypt the content
4. **Scalability**: Different user keys can be used for different users or organizations accessing the same dataset

### Implementation Notes

- User keys are never transmitted as part of the 3D Tiles dataset
- The viewing application must implement secure key storage (e.g., hardware security modules, encrypted key stores)
- Consider using key derivation functions (KDF) to generate user keys from user credentials
- Implement proper key rotation policies for long-term security

## Examples

### Tileset JSON example

```json
{
  "asset": {
    "version": "1.1"
  },
  "extensionsUsed": [
    "MAXAR_encryption_keys"
  ],
  "extensionsRequired": [
    "MAXAR_encryption_keys"
  ],
  "extensions": {
    "MAXAR_encryption_keys": {
      "keyfile": "encryption/keys.json"
    }
  },
  "geometricError": 100.0,
  "root": {
    // ...
  }
}
```

For keyfile format and examples, see the `MAXAR_buffer_encryption` glTF extension documentation.
