---
title: Android RSA公钥、私钥加密和解密
date: 2017-06-07 11:21:19
tags:
- Android
---

## 公钥私钥加密

```java
     /**
     * 公钥加密
     * @throws Exception
     */
    @Test
    public void testPublicKeyEncrypt() throws NoSuchAlgorithmException, InvalidKeySpecException, NoSuchPaddingException, InvalidKeyException, BadPaddingException, IllegalBlockSizeException {
        String pubKey = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+aOZLmOizkK325oR6SktKald6YSR8pYSFYbionJjiQKtpFjKEaAsBkiGj8WPGDMNJrYGezVvAC0PQYbxqdbjx0ybQ6JlT/nzkLIAbzQjoThS3PQDjsW/gBeELkgY4VIrqDB8VNYNohAg29zaFAP3bFkpjFwcct93c70ZvL8mz6wIDAQAB";


        String text = "test123";
        String result = "We0llfPLbCYjK6bKtauY2Ym3+vOuziObjdscv6v1uiXPDcflK81zlH2TNTLAkXzDJ9u5MgsuIp0QL6qGwFlaZU/yRV91YIJfFdOA0a1xZ+qMe5N/r6h7nCpUD+Omwc0p7pSjfkv2hUlFG062OcfVfVf2ssittW9qhLKS91WDypY=";

        // 加载公钥
        X509EncodedKeySpec data = new X509EncodedKeySpec(Base64.decode(pubKey.getBytes(), Base64.DEFAULT));
        KeyFactory factory = KeyFactory.getInstance("RSA");
        PublicKey key = factory.generatePublic(data);

        Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
        cipher.init(Cipher.ENCRYPT_MODE, key);
        byte[] encryptData = cipher.doFinal(text.getBytes());
        String encrypt = Base64.encodeToString(encryptData, Base64.DEFAULT);

        Assert.assertEquals("加密：" + encrypt, encrypt, result);


    }

    /**
     * 私钥加密
     * @throws Exception
     */
    @Test
    public void testPrivateKey() throws Exception {
        String priKey = "MIICdgIBADANBgkqhkiG9w0BAQEFAASCAmAwggJcAgEAAoGBAL5o5kuY6LOQrfbmhHpKS0pqV3phJHylhIVhuKicmOJAq2kWMoRoCwGSIaPxY8YMw0mtgZ7NW8ALQ9BhvGp1uPHTJtDomVP+fOQsgBvNCOhOFLc9AOOxb+AF4QuSBjhUiuoMHxU1g2iECDb3NoUA/dsWSmMXBxy33dzvRm8vybPrAgMBAAECgYBVGTjjzIEjz6OQV1IZ/Z5Msd5K2aOe+bKSkiwfX22MoO561urY9k8E8rSKOtYmq4mUIjFuMcWxvNcgCK5WvipbUrYaGI1wTza34ncxO7rm7/mYB1BPhX+d5lPCTNKhYix7JlDGwaC/npxQJtR9FalhxFIU+Lmr2JZN4I3swDcikQJBAPwifquvVJ75TV/Js5xGpF5E4T8t9z8O3ceQfmszglv4hXuJJLKd2UFSa2bWGP3z0x2t3qX4ZbkJ9qUrFEsUIycCQQDBVCodYi9eVXdcD0Mosv/KZjO2mx51tS6XczWfUxoyRpYdWxLfyq5vBgGEEJt4QipkgGXKnwuUppGkXBdHMdOdAkABHKHUXfyQiublcj1Bhio5ZDJeFfTOKWGe/KsiC+MaRrlH9y3bP8jyecuRc4Y+sHGQ4vBlaPgB3eJhjhQT1K3nAkB2xoa5VsFTa57RaG8SaibM6s2KuvKTzqS5V4byQ9QsX0GK95E4/QT+IOp9gNaDo+L3rArd2aj7wvpnyExk6S/hAkEAxsFtWDChCZmt62vRmz3mmrtq9scg4LplA3U0vN2glv+lc+OoRQlG0lCRFak9BH0EWqbntREeRRMn6TOI0QZKYw==";
        String text = "raetest";


        // 加载公钥
        PKCS8EncodedKeySpec data = new PKCS8EncodedKeySpec(Base64.decode(priKey.getBytes(), Base64.DEFAULT));
        KeyFactory factory = KeyFactory.getInstance("RSA");
        PrivateKey key = factory.generatePrivate(data);

        Cipher cipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
        cipher.init(Cipher.ENCRYPT_MODE, key);
        byte[] encryptData = cipher.doFinal(text.getBytes());
        String encrypt = Base64.encodeToString(encryptData, Base64.DEFAULT);

        Assert.assertEquals("加密：" + encrypt, encrypt, "12312");
    }
```


<!-- more -->