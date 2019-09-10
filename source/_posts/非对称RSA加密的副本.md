---

title: iOS非对称RSA加密
urlPath: RSA
date: 2019-06-20
updated: 2019-06-20
tag: [iOS, RSA, base64, MD5, 非对称加密]

---

### Publickey、Privatekey、Certificate

```
openssl genrsa -out private_key.pem 1024 //original key

openssl req -new -key private_key.pem -out rsaCertReq.csr 

openssl x509 -req -days 3650 -in rsaCertReq.csr -signkey private_key.pem -out rsaCert.crt

openssl x509 -outform der -in rsaCert.crt -out public_key.der // Create public_key.der For iOS

openssl pkcs12 -export -out private_key.p12 -inkey private_key.pem -in rsaCert.crt //Create private_key.p12 For iOS

openssl rsa -in private_key.pem -out rsa_public_key.pem -pubout // Create rsa_public_key.pem For Java

openssl pkcs8 -topk8 -in private_key.pem -out pkcs8_private_key.pem -nocrypt //Create pkcs8_private_key.pem For Java

```
<!-- more -->

上面七个步骤，总共生成7个文件。其中　public_key.der 和 private_key.p12 这对公钥私钥是给iOS用的，rsa_public_key.pem 和 pkcs8_private_key.pem是给JAVA用的。
它们的源都来自一个私钥：private_key.pem ， 所以iOS端加密的数据，是可以被JAVA端解密的，反过来也一样。


### iOS RSA

* head File

```
import UIKit
import CoreFoundation
import CommonCrypto

```

* Attributes

```
// padding types
let kTypeOfWrapPadding = SecPadding.PKCS1

// 公钥引用
static var publicKeyRef: SecKey?
    
// 私钥引用
static var privateKeyRef: SecKey?
```
* LoadPublicKey

```
class func loadPublicKey(_ filePath: String) -> SecKey? {
        if publicKeyRef != nil {
            publicKeyRef = nil
        }
        
        var certificateRef: SecCertificate?
        
        do {
            // 用一个.der格式证书创建一个证书对象
            let certificateData = try Data(contentsOf: URL(fileURLWithPath: filePath))
            certificateRef = SecCertificateCreateWithData(kCFAllocatorDefault, certificateData as CFData)
        } catch {
            print("file of public key is error.")
            return nil
        }
        
        // 返回一个默认 X509 策略的公钥对象
        let policyRef = SecPolicyCreateBasicX509()
        // 包含信任管理信息的结构体
        var trustRef: SecTrust?
        
        // 基于证书和策略创建一个信任管理对象
        var status = SecTrustCreateWithCertificates(certificateRef!, policyRef, &trustRef)
        if status != errSecSuccess {
            print("trust create With Certificates unsuccessfully")
            return nil
        }
        
        // 信任结果
        var trustResult = SecTrustResultType.invalid
        // 评估指定证书和策略的信任管理是否有效
        status = SecTrustEvaluate(trustRef!, &trustResult)
        
        if status != errSecSuccess {
            print("trust evaluate unsuccessfully")
            return nil
        }
        
        // 评估之后返回公钥子证书
        publicKeyRef = SecTrustCopyPublicKey(trustRef!)
        if publicKeyRef == nil {
            print("public Key create unsuccessfully")
            return nil
        }
        
        return publicKeyRef
    }
```

* LoadPrivateKey

```
  class func loadPrivateKey(_ filePath: String, _ password: String) -> SecKey? {
        if filePath.count <= 0  {
            print("path of public key is nil.")
            return nil
        }
        
        if privateKeyRef != nil {
            privateKeyRef = nil
        }
        
        var pkcs12Data: Data?
        do {
            pkcs12Data = try Data(contentsOf: URL(fileURLWithPath: filePath))
        } catch {
            print(error)
            return nil
        }
        
        let kSecImportExportPassphraseString = kSecImportExportPassphrase as String
        let options = [kSecImportExportPassphraseString: password]
        var items : CFArray?
        let status = SecPKCS12Import(pkcs12Data! as CFData, options as CFDictionary, &items)
        if status != errSecSuccess {
            print("Imports the contents of a PKCS12 formatted blob unsuccessfully")
            return nil
        }
        
        if CFArrayGetCount(items) <= 0 {
            print("the number of values currently in the array <= 0")
            return nil
        }
        
        let kSecImportItemIdentityString = kSecImportItemIdentity
        
        let dict = unsafeBitCast(CFArrayGetValueAtIndex(items, 0),to: CFDictionary.self)
        let key = Unmanaged.passUnretained(kSecImportItemIdentityString).toOpaque()
        let value = CFDictionaryGetValue(dict, key)
        let secIdentity = unsafeBitCast(value, to: SecIdentity.self)
        
        let secIdentityCopyPrivateKey = SecIdentityCopyPrivateKey(secIdentity, &privateKeyRef)
        if secIdentityCopyPrivateKey != errSecSuccess {
            print("return the private key associated with an identity unsuccessfully")
            return nil
        }
        return privateKeyRef
    }
```

* RSAEncrypt

```
public class func rsaEncrypt(_ publicKeyPath: String?, _ string: String, _ completion: ((_ encryptedString: String?) -> ())?) {
        guard let encryptData = string.data(using: String.Encoding.utf8) else {
            print("String to be encrypted is nil")
            completion?(nil)
            return
        }
        
        rsaEncrypt(publicKeyPath, encryptData) { (data) in
            guard let encryptedData = data else {
                completion?(nil)
                return
            }
            
            completion?(encryptedData.base64String)
        }
    }
```

```
public class func rsaEncrypt(_ publicKeyPath: String?, _ data: Data?, _ completion: ((_ encryptedData: Data?) -> ())?) {
        guard let pkPath = publicKeyPath,
            let pkRef = loadPublicKey(pkPath),
            let dt = data else {
                print("path of public key is nil.")
                completion?(nil)
                return
        }
        
        guard dt.count > 0 && dt.count < SecKeyGetBlockSize(pkRef) - 11
            else {
                print("The content encrypted is too large")
                completion?(nil)
                return
        }
        
        let cipherBufferSize = SecKeyGetBlockSize(pkRef)
        var encryptBytes = [UInt8](repeating: 0, count: cipherBufferSize)
        var outputSize: Int = cipherBufferSize
        let secKeyEncrypt = SecKeyEncrypt(pkRef, SecPadding.PKCS1, dt.arrayOfBytes(), dt.count, &encryptBytes, &outputSize)
        if errSecSuccess != secKeyEncrypt {
            print("decrypt unsuccessfully")
            completion?(nil)
            return
        }
        
        completion?(Data(bytes: UnsafePointer<UInt8>(encryptBytes), count: outputSize))
    }
```

* RSADecrypt

```
public class func rsaDecrypt(_ privateKeyPath: String?, _ password: String, _ string: String?, _ completion: ((_ encryptedString: String?) -> ())?) {
        guard let str = string,
            let decryptData = Data(base64Encoded: str) else {
                print("String to be decrypted is nil")
                completion?(nil)
                return
        }
        
        rsaDecrypt(privateKeyPath, password, decryptData) { (data) in
            guard let decryptedData = data else {
                completion?(nil)
                return
            }
            
            completion?(String(data: decryptedData, encoding: String.Encoding.utf8))
        }
    }
```
```
 public class func rsaDecrypt(_ privateKeyPath: String?, _ password: String, _ data: Data?, _ completion: ((_ encryptedData: Data?) -> ())?) {
        guard let pkPath = privateKeyPath,
            let pkRef = loadPrivateKey(pkPath, password),
            let dt = data else {
                print("path of private key is nil.")
                completion?(nil)
                return
        }
        
        let cipherBufferSize = SecKeyGetBlockSize(pkRef)
        let keyBufferSize = dt.count
        if keyBufferSize > cipherBufferSize {
            print("The content decrypted is too large")
            completion?(nil)
            return
        }
        
        var decryptBytes = [UInt8](repeating: 0, count: cipherBufferSize)
        var outputSize = cipherBufferSize
        let status = SecKeyDecrypt(pkRef, SecPadding.PKCS1, dt.arrayOfBytes(), dt.count, &decryptBytes, &outputSize)
        if errSecSuccess != status {
            print("decrypt unsuccessfully")
            completion?(nil)
            return
        }
        
        completion?(Data(bytes: UnsafePointer<UInt8>(decryptBytes), count: outputSize))
    }
```
### Base64

```
extension Data {
    
    /// Data to base64 String
    public var base64String: String {
        return base64EncodedString(options: NSData.Base64EncodingOptions())
    }
    
    /// Array of UInt8
    public func arrayOfBytes() -> [UInt8] {
        let count = self.count / MemoryLayout<UInt8>.size
        var bytesArray = [UInt8](repeating: 0, count: count)
        (self as NSData).getBytes(&bytesArray, length:count * MemoryLayout<UInt8>.size)
        return bytesArray
    }
}
```
### MD5
* String


```
 public func md5() -> String {
        let str = self.cString(using: String.Encoding.utf8)
        let strLen = CUnsignedInt(self.lengthOfBytes(using: String.Encoding.utf8))
        let digestLen = Int(CC_MD5_DIGEST_LENGTH)
        let result = UnsafeMutablePointer<UInt8>.allocate(capacity: 16)
        CC_MD5(str!, strLen, result)
        let hash = NSMutableString()
        for i in 0 ..< digestLen {
            hash.appendFormat("%02x", result[i])
        }
        free(result)
        return String(format: hash as String)
    }
```

* File

```
public func md5_File() -> String? {
        guard let fileHandle = FileHandle(forReadingAtPath: self) else {
            return nil
        }
        
        let ctx = UnsafeMutablePointer<CC_MD5_CTX>.allocate(capacity: MemoryLayout<CC_MD5_CTX>.size)
        
        CC_MD5_Init(ctx)
        
        var done = false
        
        while !done {
            let fileData = fileHandle.readData(ofLength: 256)
            fileData.withUnsafeBytes {(bytes: UnsafePointer<CChar>) -> () in
                /// Use `bytes` inside this closure
                CC_MD5_Update(ctx, bytes, CC_LONG(fileData.count))
            }
            
            if fileData.count == 0 {
                done = true
            }
        }
        
        let digest = Int(CC_MD5_DIGEST_LENGTH)
        let result = UnsafeMutablePointer<CUnsignedChar>.allocate(capacity: digest)
        CC_MD5_Final(result, ctx);
        
        var hash = ""
        for i in 0..<digest {
            hash +=  String(format: "%02x", (result[i]))
        }
        
        free(result)
        free(ctx)
        
        return hash;
    }
```

### RSA分段加密
* RSAEncrypt

```
 public class func rsaEncrypt(_ publicKeyPath: String?, _ string: String, _ completion: ((_ encryptedString: String?) -> ())?) {
        guard let encryptData = string.data(using: String.Encoding.utf8) else {
            print("String to be encrypted is nil")
            completion?(nil)
            return
        }
        
        rsaEncrypt(publicKeyPath, encryptData) { (data) in
            guard let encryptedData = data else {
                completion?(nil)
                return
            }
            
            completion?(encryptedData.base64String)
        }
    }
```
```
 public class func rsaEncrypt(_ publicKeyPath: String?, _ data: Data?, _ completion: ((_ encryptedData: Data?) -> ())?) {
        guard let pkPath = publicKeyPath,
            let pkRef = loadPublicKey(pkPath),
            let dt = data else {
                print("path of public key is nil.")
                completion?(nil)
                return
        }
        let blockLen =  SecKeyGetBlockSize(pkRef)
        var outBuf = [UInt8](repeating: 0, count: blockLen)
        var outBufLen:Int = blockLen
        var index = 0
        let totalLen = dt.count
        let resData = NSMutableData()
        
        while index < totalLen {
            var curDataLen = totalLen - index
            if curDataLen > blockLen - 11 {
                curDataLen = blockLen - 11
            }
            let curData: NSData = dt.subdata(in: Range(NSRange(location: index, length: curDataLen))!) as NSData
            let status: OSStatus = SecKeyEncrypt(pkRef, SecPadding.PKCS1, curData.bytes.assumingMemoryBound(to: UInt8.self), curData.length, &outBuf, &outBufLen)
            
            if status == noErr {
                resData.append(outBuf, length: outBufLen)
            }else{
                print("encrypt status = \(status)")
            }
            
            index += curDataLen
        }
        completion?(resData as Data)
```

* RSADecrypt

```
  public class func rsaDecrypt(_ privateKeyPath: String?, _ password: String, _ string: String?, _ completion: ((_ encryptedString: String?) -> ())?) {
        guard let str = string,
            let decryptData = Data(base64Encoded: str) else {
                print("String to be decrypted is nil")
                completion?(nil)
                return
        }
        
        rsaDecrypt(privateKeyPath, password, decryptData) { (data) in
            guard let decryptedData = data else {
                completion?(nil)
                return
            }
            
            completion?(String(data: decryptedData, encoding: String.Encoding.utf8))
        }
    }
```
```
 public class func rsaDecrypt(_ privateKeyPath: String?, _ password: String, _ data: Data?, _ completion: ((_ encryptedData: Data?) -> ())?) {
        guard let pkPath = privateKeyPath,
            let pkRef = loadPrivateKey(pkPath, password),
            let dt = data else {
                print("path of private key is nil.")
                completion?(nil)
                return
        }
        let blockLen = SecKeyGetBlockSize(pkRef)
        let outBuf = UnsafeMutablePointer<UInt8>.allocate(capacity: blockLen)
        defer {
            outBuf.deallocate()
        }
        var outBufLen: Int = blockLen
        var index = 0
        let totalLen = dt.count
        let resData = NSMutableData()
        
        while index < totalLen {
            var curDataLen = totalLen - index
            if curDataLen  > blockLen {
                curDataLen = blockLen
            }
            let curData: Data = dt.subdata(in: index ..< index + curDataLen)
            var status:OSStatus = noErr
            curData.withUnsafeBytes { (bytes:UnsafePointer<UInt8>) in
                //print(bytes)
                status = SecKeyDecrypt(pkRef, SecPadding.PKCS1, bytes, curData.count, outBuf, &outBufLen)
            }
            
            if status == noErr {
                resData.append(outBuf, length: outBufLen)
            }else{
                print("decrypt status = \(status)")
            }
            index += curDataLen
        }
        completion?(resData as Data)
```