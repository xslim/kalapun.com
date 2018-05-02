---
published: true
title: "Ditching OpenSSL - reimplementing AES-CCM on iOS"
---

The project I'm working on to safely transmit vulnerable data over public networks uses a specially composed message with encrypted data and key.

## The why

Here is the process of message creation:
1. A session key and nonce are created from random bits of data
2. Vulnerable data is encrypted using *AES-CCM*
3. A session key is encrypted with a public certificate using *RSA*
4. A message is composed by combining the encrypted key, encrypted data, authentication tag and nonce.

This process is quite common in enterprise, banking and other applications.

In case of implementing it on iOS we can find that the native `CommonCrypto` framework has *AES*, but misses the block cipher mode, the *CCM*. The `Security` framework also misses straight-forward method for importing public key from mantissa and exponent. 

Many choose to use OpenSSL library, which has the above functionality built in. The downside is that it takes around 10 megabytes of additional data to link into your project and many do not bother updating it.

The other way is to spend some time doing a bit of research, code the missing functionality and forget the OpenSSL madness.

## Possible solutions
* swap OpenSSL to another big fat library
* use [VPCCMCrypt](https://github.com/billp/VPCCMCrypt) - has the AES-CCM implementation, but not the RSA; not well tested; may have unsuspected functionality.
* google for good C code of block cipher and convert it to ones needs.

## Implementing AES-CCM
* Code is in [TKAESCCMCryptor.m](https://github.com/xslim/TKCryptor/blob/master/TKCryptor/TKAESCCMCryptor.m) file
* Some code parts are from [tinydtls](https://github.com/cetic/tinydtls/)
* The main C function is `ccm_encrypt_message`
* The `aes_encrypt` function uses the `CCCrypt` in *AES-ECB* mode for encrypting the block


## Implementing RSA
* * Code is in [TKRSACryptor.m](https://github.com/xslim/TKCryptor/blob/master/TKCryptor/TKRSACryptor.m) file
* Some code from [iphonelib](https://github.com/meinside/iphonelib) is used
* First, the RSA Public key is generated with `+ (NSData *)generateRSAPublicKeyWithModulus:(NSData*)modulus exponent:(NSData*)exponent`
* Then, it is saved in a Keychain for later re-use
* If the fingerprint is calculated with SHA1 native method, note that sometimes it fails :(

## Testing
The most difficult part of code is the Block cipher. If it's not implemented correctly, the encryption will not fail, it will  just be wrong. Thus testing it was quite necessary.

The common way to test encryption algorithms is using test vectors that are available in appendix in RFC. In case of [RFC-3610](https://github.com/xslim/TKCryptor/blob/master/rfc3610.txt#L549) they come in blocks, where every block contains input data in hex; parameters, like tag length; data from some steps; and finally, the result data. 

The test suit may consists of 2 parts:
 1. extract the data from RFC document and save it in JSON - the small [ruby script](https://github.com/xslim/TKCryptor/blob/master/Tests/extract_test_vectors.rb) with `regex` parsing
 2. parse the JSON, feed the vectors in the Encryption function and compare the result - which is done in the [Xcode test case](https://github.com/xslim/TKCryptor/blob/master/Tests/Tests/AESCCMTest.m#L23)

The final piece would be to use Continuous Integration to detect any regression that might occur in the future code edits. TravisCI is one easy way to do it - just create `.travis` file and enable TravisCI on the project. 

``` yaml
language: objective-c
before_install:
  - brew update
  - if brew outdated | grep -qx xctool; then brew upgrade xctool; fi
  - cd Tests && ./extract_test_vectors.rb && pod install
script:
- xctool test -workspace TKCryptor.xcworkspace -scheme Tests -sdk iphonesimulator ONLY_ACTIVE_ARCH=NO
```

The next commit will trigger the build, and we can see the builds are passing. 

The final project can be found at [xslim/TKCryptor](https://github.com/xslim/TKCryptor)
