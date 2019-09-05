## 概述
- 主页：[Apache Commons Codec](	https://commons.apache.org/proper/commons-codec/)
- 介绍：主要提供了通用的编解码文本或和二进制流的工具，例如，Base64、十六进制转换（Hex）、语音库（Phonetic）及URL编解码等
- 主要内容：参考[User Guide](https://commons.apache.org/proper/commons-codec/userguide.html)
    - 二进制编码器：较为重要的是`Base64`及`Hex`
    - 摘要工具：`DigestUtils`，主要实现了基本的消息摘要算法及密码hash功能
    - 语言工具
    - 网络工具：例如，`URLCodec`实现了`url-form-encoded`编码模式
- Maven引用
```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
    <version>1.13</version>
</dependency>
```

## DigestUtils应用
项目实战中，我们需要MD5/BASE64/SHAXXX等的编解码，DigestUtils已经封装了相关方法，具体如下：
```java
import java.util.Base64;
import org.apache.commons.codec.digest.DigestUtils;

@Override
public String encryptPassword(String passWord, PasswordEncryptType passwordEncryptType) {
    if (passwordEncryptType == null) {
        return passWord;
    }
    String encryptPWD = null;
    switch (passwordEncryptType) {
        case MD5:
            encryptPWD = DigestUtils.md5Hex(passWord);
            break;
        case BASE64:
            encryptPWD = Base64.getEncoder().encodeToString(passWord.getBytes());
            break;
        case SHA256:
            encryptPWD = DigestUtils.sha256Hex(passWord);
            break;
        case SHA512:
            encryptPWD = DigestUtils.sha512Hex(passWord);
            break;
        default:
            break;
    }
    return encryptPWD == null ? passWord : encryptPWD;
}
```