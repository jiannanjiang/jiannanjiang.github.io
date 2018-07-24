# Openssl 命令



## 生成密钥

私钥

`openssl genrsa -out rsa_weixiaobao_private_key.pem 1024`

公钥

`openssl rsa -in rsa_weixiaobao_private_key.pem -pubout -out rsa_weixiaobao_public_key.pem`



## 非对称加密

* 私钥用来解密数据

* 共钥用来加密数据

* 私钥用来生成签名

* 公钥用来验证签名

  ​

