---
layout: post
title: "암호화 모듈과 API 사용 방법(Java, Python, AES, RSA, SHA-256)"
tags: [Security, Encryption, Java, Python, API, AES, RSA, SHA-256]
---

# Intro

안녕하세요, Noah입니다.

오늘은 암호화 모듈과 이를 활용한 Java와 Python API 사용 방법에 대해 다뤄보려고 합니다. 

이번 글에서는 암호화 모듈들의 사용처부터 이를 Java와 Python에서 어떻게 사용하는지에 대해 예시 코드와 함께 설명드리겠습니다.

이제 시작해보겠습니다!

<br/><br/><br/><br/>

---
# 목차
1. [암호화 기초 지식](#암호화와-관련된-기초-지식)
   - [암호화의 필요성](#1-암호화의-필요성)
   - [단방향 암호화 vs. 양방향 암호화](#2-단방향-암호화-vs-양방향-암호화)
   - [주요 암호화 알고리즘](#3-주요-암호화-알고리즘)
   - [암호화 방식의 선택 기준](#4-암호화-방식의-선택-기준)
   - [암호화 사례](#5-암호화-사례)
2. [Java에서 사용하는 암호화 모듈](#1-java에서-암호화-모듈을-지원하는-api)
   - [AES 암호화 - Java API](#1-1-aes-암호화---java-api)
   - [RSA 암호화 - Java API](#1-2-rsa-암호화---java-api)
   - [SHA-256 해싱 - Java API](#1-3-sha-256-해싱---java-api)
3. [Python에서 사용하는 암호화 모듈](#2-python에서-암호화-모듈을-지원하는-api)
   - [AES 암호화 - Python API](#2-1-aes-암호화---python-api-pycryptodome)
   - [RSA 암호화 - Python API](#2-2-rsa-암호화---python-api-pycryptodome)
   - [SHA-256 해싱 - Python API](#2-3-sha-256-해싱---python-api-hashlib)
4. [Outro](#outro)
<br/><br/><br/><br/>

---
# 암호화와 관련된 기초 지식
## 1. 암호화의 필요성
암호화는 민감한 정보를 안전하게 보호하기 위한 핵심 기술입니다. 

사용자의 개인정보, 금융 정보, 비밀문서 등 보안이 중요한 데이터는 암호화를 통해 무단 접근과 데이터 유출을 방지할 수 있습니다.

예를 들어, 사용자가 웹사이트에 비밀번호를 입력하면 그 데이터는 암호화되어 데이터베이스에 저장됩니다. 이는 데이터 유출 사고가 발생했을 때도 암호화된 형태로 정보가 저장되어 있기에, 공격자가 비밀번호를 그대로 읽을 수 없도록 하는 역할을 합니다.
<br/><br/><br/>


## 2. 단방향 암호화 vs. 양방향 암호화
암호화 방식은 크게 **단방향 암호화**와 **양방향 암호화**로 나뉩니다.

### 2-1. 단방향 암호화 (Hashing)
단방향 암호화는 암호화된 데이터가 원래 데이터로 복호화될 수 없도록 설계된 방식입니다. 해싱 알고리즘이 주로 사용되며, 주로 비밀번호 저장에 적합합니다. 

비밀번호를 해시값으로 변환하여 데이터베이스에 저장하고, 로그인 시에도 입력된 비밀번호를 해시화하여 기존 해시값과 비교합니다. 

해싱은 **SHA (Secure Hash Algorithm)**, **MD5 (Message Digest Algorithm)** 같은 알고리즘을 통해 구현할 수 있습니다.

> **SHA256**: 취약점이 발견된 MD5와 다르게 현재 많은 시스템에서 사용되는 해시 알고리즘으로, 데이터의 일관성과 무결성을 보장합니다. 원본 데이터가 조금만 변경되어도 완전히 다른 해시값을 반환하기 때문에 데이터 위변조를 쉽게 감지할 수 있습니다.
> 
> **사용 사례**: SHA256과 같은 해시는 비밀번호 저장, 데이터 무결성 확인, 블록체인 거래 기록 검증 등에 활용됩니다.

#### 단방향 암호화의 특징
- **복호화 불가능**: 단방향 암호화는 복호화 과정이 존재하지 않으므로, 데이터 자체를 노출하지 않고 비교할 수 있습니다.
- **동일 입력 → 동일 출력**: 동일한 입력값은 언제나 같은 해시값을 생성하므로, 입력값을 알 수 없는 경우 해시값만으로는 원본을 추측할 수 없습니다.
- **무결성 검증**: 파일의 원본 여부를 확인하는 데 사용할 수 있습니다. 예를 들어, 파일이 안전한 상태로 유지되었는지 검증할 때 해시값을 비교하여 파일의 무결성을 확인합니다.
<br/><br/>

### 2-2. 양방향 암호화
양방향 암호화는 암호화와 복호화가 모두 가능한 방식으로, 데이터를 인코딩한 후 다시 원본 데이터로 복구할 수 있습니다. 일반적으로 대칭키 암호화와 비대칭키 암호화 방식이 있습니다.

> **대칭키 암호화**(Symmetric Encryption): 하나의 동일한 키를 사용해 암호화와 복호화를 모두 수행하는 방식입니다. **AES (Advanced Encryption Standard)**와 같은 알고리즘이 대표적인 예입니다. AES는 속도와 안전성을 모두 고려해 설계된 대칭키 암호화 알고리즘으로, 데이터 전송 시 자주 사용됩니다.
> 
> **사용 사례**: 대칭키 암호화는 서버와 클라이언트 간의 안전한 데이터 전송, 파일 암호화, 디스크 암호화 등에 널리 사용됩니다.

> **비대칭키 암호화**(Asymmetric Encryption): 서로 다른 두 개의 키(공개키와 개인키)를 사용하는 방식입니다. **RSA** 알고리즘이 대표적이며, 데이터 암호화 시 공개키로 암호화한 데이터를 개인키를 가진 사용자만 복호화할 수 있도록 설계되어 보안성이 높습니다.
> 
> **사용 사례**: 비대칭키 암호화는 주로 전자 서명, SSL/TLS 인증서, 전자 메일 보안 등에 활용됩니다.

#### 양방향 암호화의 특징
- **복호화 가능**: 양방향 암호화는 복호화 과정이 있어 원본 데이터를 다시 복구할 수 있습니다.
- **대칭키 vs. 비대칭키**: 대칭키 방식은 동일한 키를 공유해 빠르게 암호화/복호화가 가능하지만, 키 분배가 어려울 수 있습니다. 반면 비대칭키 방식은 보안성이 더 높은 대신, 대칭키보다 암호화/복호화 속도가 느립니다.
- **공개키와 개인키의 활용**: 비대칭키는 공개키로 데이터를 암호화하고, 오직 개인키로만 복호화할 수 있어, 송신자가 공개키를 알고 있더라도 수신자의 개인키 없이는 데이터를 해독할 수 없습니다.
<br/><br/><br/>

## 3. 주요 암호화 알고리즘
- **AES (Advanced Encryption Standard)**: 현재 많은 시스템에서 사용되는 대칭키 암호화 방식으로, 보안성과 성능이 우수해 암호화 표준으로 자리 잡고 있습니다. 128비트, 192비트, 256비트 키 길이를 지원하며, 데이터 전송과 저장 시 널리 사용됩니다.
- **RSA (Rivest-Shamir-Adleman)**: 비대칭키 암호화 알고리즘 중 하나로, 큰 소수의 곱을 이용해 공개키와 개인키를 생성합니다. 주로 데이터 보호와 전자 서명에 사용됩니다.
- **SHA-256**: 단방향 암호화 방식의 해시 알고리즘 중 하나로, 해시 값을 계산하여 원본 데이터가 변경되지 않았음을 확인할 수 있습니다. 블록체인 등에서 무결성 검증에 많이 사용됩니다.
<br/><br/><br/>

## 4. 암호화 방식의 선택 기준
- **보안성**: 암호화가 필요한 데이터의 중요도에 따라 안전성을 고려한 암호화 방식 선택이 필요합니다. 비밀번호와 같은 민감한 정보는 해시 알고리즘을 통해 저장하고, 실시간 데이터 전송에는 대칭키 암호화가 적합할 수 있습니다.
- **속도**: 대칭키 방식이 비대칭키 방식보다 빠르므로, 실시간 처리가 필요한 경우 대칭키 암호화를, 고도의 보안이 필요한 경우 비대칭키 암호화를 사용합니다.
- **용도**: 사용 목적에 따라 적합한 암호화 방식을 선택합니다. 파일 암호화나 데이터베이스 보호가 필요하면 AES 같은 대칭키 암호화를, 인증이 필요하면 RSA와 같은 비대칭키 암호화를 사용합니다.
<br/><br/><br/>

## 5. 암호화 사례
- **HTTPS**: 웹사이트와 사용자 간 데이터가 안전하게 전송되도록 하기 위해 SSL/TLS 프로토콜을 사용하는 HTTPS에서는 RSA를 사용해 인증서를 발급받고 대칭키를 교환하여 이후 통신을 보호합니다.
- **블록체인**: 블록체인에서 사용되는 트랜잭션 해시는 SHA256과 같은 해시 알고리즘을 사용해 데이터의 무결성을 보장합니다.
- **전자 서명**: 비대칭키 암호화 방식을 사용해 서명자가 본인의 개인키로 서명하고, 이를 검증할 때는 공개키를 사용하는 방식입니다.
<br/><br/><br/><br/>

---
# Java와 Python에서 사용하는 암호화 모듈
## 1. Java에서 암호화 모듈을 지원하는 API
Java는 암호화를 위해 **javax.crypto**와 **java.security** 패키지를 제공합니다.<br/>
이를 통해 AES, RSA, SHA-256 같은 암호화 알고리즘을 구현할 수 있습니다.

### 1-1. AES 암호화 - Java API
AES는 대칭키 암호화 알고리즘으로, Java의 **javax.crypto** 패키지에서 지원합니다. <br/>
Java에서는 `Cipher`와 `KeyGenerator`, `SecretKeySpec` 클래스를 사용하여 AES 암호화 및 복호화를 쉽게 구현할 수 있습니다.
<br/><br/>

#### 주요 클래스 및 메서드
- **Cipher**: 암호화 및 복호화 연산을 수행하는 클래스입니다.
- **KeyGenerator**: AES 암호화에 사용할 비밀키를 생성합니다.
- **SecretKeySpec**: 생성된 키를 AES 알고리즘에 맞게 변환하는 클래스입니다.
<br/><br/>

#### 예제 코드 (Java AES)
```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

public class AESExample {
    public static void main(String[] args) throws Exception {
        // 키 생성
        KeyGenerator keyGen = KeyGenerator.getInstance("AES");
        keyGen.init(128); // 키 길이 설정 (128, 192, 256 가능)
        SecretKey secretKey = keyGen.generateKey();

        // 암호화
        Cipher cipher = Cipher.getInstance("AES");
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        String plainText = "Hello, World!";
        byte[] encryptedBytes = cipher.doFinal(plainText.getBytes());
        String encryptedText = Base64.getEncoder().encodeToString(encryptedBytes);

        System.out.println("Encrypted Text: " + encryptedText);

        // 복호화
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(encryptedText));
        String decryptedText = new String(decryptedBytes);

        System.out.println("Decrypted Text: " + decryptedText);
    }
}
```
<br/><br/><br/>

### 1-2. RSA 암호화 - Java API
RSA는 비대칭키 암호화 알고리즘으로, Java의 **java.security** 패키지에서 지원합니다. <br/>
`KeyPairGenerator`와 `Cipher` 클래스를 사용하여 RSA 암호화와 복호화를 수행할 수 있습니다.
<br/><br/>

#### 주요 클래스 및 메서드
- **KeyPairGenerator**: 공개키와 개인키 쌍을 생성하는 클래스입니다.
- **Cipher**: RSA 암호화와 복호화를 수행합니다.
<br/><br/>

#### 예제 코드 (Java RSA)
```java
import javax.crypto.Cipher;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.util.Base64;

public class RSAExample {
    public static void main(String[] args) throws Exception {
        // RSA 키 쌍 생성
        KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
        keyGen.initialize(2048);
        KeyPair keyPair = keyGen.generateKeyPair();

        // 암호화
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, keyPair.getPublic());
        String plainText = "Hello, World!";
        byte[] encryptedBytes = cipher.doFinal(plainText.getBytes());
        String encryptedText = Base64.getEncoder().encodeToString(encryptedBytes);

        System.out.println("Encrypted Text: " + encryptedText);

        // 복호화
        cipher.init(Cipher.DECRYPT_MODE, keyPair.getPrivate());
        byte[] decryptedBytes = cipher.doFinal(Base64.getDecoder().decode(encryptedText));
        String decryptedText = new String(decryptedBytes);

        System.out.println("Decrypted Text: " + decryptedText);
    }
}
```
<br/><br/><br/>

### 1-3. SHA-256 해싱 - Java API
SHA-256은 단방향 해시 함수로, **java.security** 패키지의 `MessageDigest` 클래스를 통해 사용할 수 있습니다. <br/>
해시 함수는 복호화가 불가능한 단방향 변환을 제공하여 주로 데이터 무결성 검증에 사용됩니다.
<br/><br/>

#### 주요 클래스 및 메서드
- **MessageDigest**: SHA-256 해싱을 수행하는 클래스입니다. `digest()` 메서드로 입력 데이터를 해싱하여 고유한 해시값을 생성할 수 있습니다.
<br/><br/>

#### 예제 코드 (Java SHA-256)
```java
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.Base64;

public class SHA256Example {
    public static void main(String[] args) throws NoSuchAlgorithmException {
        String plainText = "Hello, World!";
        MessageDigest digest = MessageDigest.getInstance("SHA-256");
        byte[] hash = digest.digest(plainText.getBytes());
        String hashString = Base64.getEncoder().encodeToString(hash);

        System.out.println("SHA-256 Hash: " + hashString);
    }
}
```
<br/><br/><br/><br/>

---
## 2. Python에서 암호화 모듈을 지원하는 API
Python에서는 기본 라이브러리 대신 **PyCryptodome**을 사용하여 AES, RSA, SHA-256을 구현합니다. <br/>
PyCryptodome은 Python에서 강력한 암호화 기능을 제공하며, 설치는 `pip install pycryptodome`으로 가능합니다.
<br/><br/><br/>

### 2-1. AES 암호화 - Python API (PyCryptodome)
AES는 대칭키 암호화 방식으로, PyCryptodome의 `Crypto.Cipher.AES` 모듈을 통해 구현할 수 있습니다. <br/>
Python에서는 주로 **CBC (Cipher Block Chaining)** 모드를 사용하여 암호화합니다.
<br/><br/>

#### 주요 클래스 및 메서드
- **AES**: AES 암호화와 복호화를 수행하는 클래스입니다.
- **get_random_bytes**: 키와 초기화 벡터(IV)를 생성하는 데 사용됩니다.
- **pad/unpad**: 데이터 길이를 맞추기 위해 패딩을 추가/제거하는 함수입니다.
<br/><br/>

#### 예제 코드 (Python AES)
```python
from Crypto.Cipher import AES
from Crypto.Random import get_random_bytes
from Crypto.Util.Padding import pad, unpad

# AES 키 및 초기화 벡터 생성
key = get_random_bytes(16)
iv = get_random_bytes(16)

# 암호화
cipher = AES.new(key, AES.MODE_CBC, iv)
plain_text = "Hello, World!".encode()
cipher_text = cipher.encrypt(pad(plain_text, AES.block_size))

print("Encrypted Text:", cipher_text)

# 복호화
cipher = AES.new(key, AES.MODE_CBC, iv)
decrypted_text = unpad(cipher.decrypt(cipher_text), AES.block_size).decode()

print("Decrypted Text:", decrypted_text)
```
<br/><br/><br/>

### 2-2. RSA 암호화 - Python API (PyCryptodome)
RSA는 비대칭키 암호화 방식으로, PyCryptodome의 `Crypto.PublicKey.RSA`와 `Crypto.Cipher.PKCS1_OAEP` 모듈을 통해 구현할 수 있습니다. <br/>
공개키와 개인키를 생성하여 암호화와 복호화에 사용합니다.
<br/><br/>

#### 주요 클래스 및 메서드
- **RSA.generate**: 공개키와 개인키 쌍을 생성합니다.
- **PKCS1_OAEP**: RSA 암호화 및 복호화를 수행하는 클래스입니다.
<br/><br/>

#### 예제 코드 (Python RSA)
```python
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_OAEP
import base64

# RSA 키 쌍 생성
key = RSA.generate(2048)
public_key = key.publickey()

# 암호화
cipher = PKCS1_OAEP.new(public_key)
plain_text = "Hello, World!".encode()
cipher_text = cipher.encrypt(plain_text)
encoded_cipher_text = base64.b64encode(cipher_text)

print("Encrypted Text:", encoded_cipher_text)

# 복호화
cipher = PKCS1_OAEP.new(key)
decoded_cipher_text = base64.b64decode(encoded_cipher_text)
decrypted_text = cipher.decrypt(decoded_cipher_text).decode()

print("Decrypted Text:", decrypted_text)
```
<br/><br/><br/>

### 2-3. SHA-256 해싱 - Python API (hashlib)
SHA-256은 단방향 해시 알고리즘으로, Python의 내장 라이브러리인 `hashlib`을 통해 구현할 수 있습니다. <br/>
`hashlib` 모듈의 `sha256()` 함수를 사용하여 문자열을 해싱할 수 있습니다.
<br/><br/>

#### 주요 함수 및 메서드
- **hashlib.sha256**: SHA-256 해싱을 수행하는 함수로, 입력 데이터에 대해 해시 값을 계산합니다.
<br/><br/>

#### 예제 코드 (Python SHA-256)
```python
import hashlib
import base64

plain_text = "Hello, World!"
hash_object = hashlib.sha256(plain_text.encode())
hash_string = base64.b64encode(hash_object.digest()).decode()

print("SHA-256 Hash:", hash_string)
```
<br/><br/><br/><br/>

---
# Outro
오늘은 암호화의 기본 개념부터 Java와 Python에서 **AES**, **RSA**, **SHA-256** 등 암호화 모듈을 활용하는 방법에 대해 알아보았습니다. 

각각의 API를 활용해 보면서 암호화의 중요성과 적용 방법을 이해하셨길 바랍니다. 

궁금한 점이나 더 알고 싶은 내용이 있으시면 댓글로 남겨주세요!

긴 글 읽어주셔서 감사합니다!