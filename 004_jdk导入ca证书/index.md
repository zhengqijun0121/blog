# JDK 导入 CA 证书


## 显示的错误信息

```text
javax.net.ssl.SSLHandshakeException: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target
```

导致这个错误的原因是没有有效的证书，因此导入相应的证书就可以正常访问了。

证书可以通过网站 -> Certificate -> Details -> Export 获取。

## 证书管理

1. 导入证书

```shell
keytool -import -keystore ~/Opt/jdk1.8.0_181/jre/lib/security/cacerts -alias ca -storepass changeit -keypass changeit -file ~/Downloads/ca.crt
```

2. 删除证书

```shell
keytool -delete -keystore ~/Opt/jdk1.8.0_181/jre/lib/security/cacerts -alias ca -storepass changeit
```

3. 查看证书

```shell
keytool -list -keystore ~/Opt/jdk1.8.0_181/jre/lib/security/cacerts -alias ca
```


