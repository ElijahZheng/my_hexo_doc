title: Java 读取公钥、私钥，SHA256算法加签、验签
author: Elijah Zheng
tags:
  - java
categories:
  - java
date: 2019-12-05 15:27:00
---
最近在做利用私钥加签，和对加签串利用公钥验签，作如下总结：
<!--more-->

## 读取公钥

``` java
public static PublicKey getPublicKey() {
    try {
        File file = new File("公钥 cer 路径");
        InputStream inStream = new FileInputStream(file);
        // 创建X509工厂类
        CertificateFactory cf = CertificateFactory.getInstance("X.509");
        // 创建证书对象
        X509Certificate cert = (X509Certificate) cf
                .generateCertificate(inStream);
        inStream.close();
        PublicKey publicKey = cert.getPublicKey();
        return publicKey;

    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

## 读取私钥
``` java
public static PrivateKey getPrivateKey() {
    try {
        String keyStorePath = "私钥 pfx 地址";
        String password = "密码";

        // 实例化密钥库，默认JKS类型
        KeyStore ks = KeyStore.getInstance("PKCS12");
        // 获得密钥库文件流
        FileInputStream is = new FileInputStream(keyStorePath);
        // 加载密钥库
        ks.load(is, password.toCharArray());
        // 关闭密钥库文件流
        is.close();

        //私钥
        Enumeration aliases = ks.aliases();
        String keyAlias = null;
        if (aliases.hasMoreElements()){
            keyAlias = (String)aliases.nextElement();
            System.out.println("p12's alias----->"+keyAlias);
        }
        return (PrivateKey) ks.getKey(keyAlias, password.toCharArray());
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

## 加签
``` java
/**
* 私钥加签
* @param unsignstr 需要加签的拼接串
* @return 加签串
*/
public static String GetSign(String unsignstr){
    String signature = null;
    byte[] signed = null;
    try {
        PrivateKey privateKey = getPrivateKey();
        String privateKeyValue = Base64.getEncoder().encodeToString(privateKey.getEncoded());
        System.out.println("私钥------------->" + privateKeyValue);
        Signature Sign = Signature.getInstance(SIGNATURE_ALGORITHM);
        Sign.initSign(privateKey);

        System.out.println(unsignstr);
        byte[] outputDigest_sign = unsignstr.getBytes();
        Sign.update(outputDigest_sign);
        signed = Sign.sign();
        signature = Base64.getEncoder().encodeToString(signed);
        logger.info(signature);
        System.out.println(signature);

    } catch (Exception e) {
        e.printStackTrace();
    }
    return signature;
}
```

## 验签
``` java
/**
* 公钥验签
* @param requsetBody 验签报文
* @param signature 验签串
*/
public static void read_cer_and_verify_sign(String requsetBody, String signature) {
    String filePath = "公钥路径";
    X509Certificate cert = null;
    try {
        CertificateFactory cf = CertificateFactory.getInstance("X.509");
        cert = (X509Certificate) cf
                .generateCertificate(new FileInputStream(new File(filePath)));
        PublicKey publicKey = cert.getPublicKey();
        String publicKeyString = new String(Base64.getEncoder().encode(publicKey
                .getEncoded()));
        System.out.println("-----------------公钥--------------------");
        System.out.println(publicKeyString);
        System.out.println("-----------------公钥--------------------");
        Signature verifySign = Signature.getInstance(SIGNATURE_ALGORITHM);
        verifySign.initVerify(publicKey);
        // 用于验签的数据
        System.out.println("requestBody is " + requsetBody);
        verifySign.update(requsetBody.getBytes());
        boolean flag = verifySign.verify(Base64.getDecoder().decode(signature));
        System.out.println("verifySign is " + flag);
    } catch (InvalidKeyException e) {
        e.printStackTrace();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    } catch (SignatureException e) {
        e.printStackTrace();
    } catch (CertificateException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    }
}
```