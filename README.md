# 前文

本项目由`lpilp`大佬以下项目fork而来

[https://github.com/lpilp/phpsm2sm3sm4](https://github.com/lpilp/phpsm2sm3sm4)

### 注意点

* 本项目是为了适配`php5.6`而做的修改，`php:7.2`及以上不建议使用
* 由于官方打包的`windows`版`php`都是`32`位的，所以在本项目的`位运算`中，会导致溢出`PHP_INT_MAX`内核常量

#### 如何查看php是否32位

1. 输出`PHP_INT_SIZE`，如果返回是 4 就是 32 位的，如果返回是 8 就是 64 位的
2. 输出`PHP_INT_MAX`， 64 位的会返回 `9223372036854775807`

#### 如何避免`32`位的`位预算溢出`问题?有两种办法

1. 改换`64`位的`php`(线上项目估计换不了， `windows`也难以自己编译`64`位的`php`)
2. 将`位运算`改成使用`gmp`扩展，如以下代码示例

```php
//以下两句代码是一样的效果
$r = gmp_intval(gmp_div(128, gmp_pow(2, 3)));//$r = 16

$r = 128 >> 3;//$r = 16
```

------

# php sm2 sm3 sm4 国密算法整理
* php版本的国密sm2的签名算法，非对称加解密算法（非对称加密刚上线，目前测试无问题，不能保证兼容其他语言，有问题可以提issues），sm3的hash,  sm4的对称加解密，要求PHP７，打开gmp支持
* 目前如果服务器配套的使用的是openssl 1.1.1x, 目前到1.1.1.l(L) ,sm3,sm4都可以直接用openssl_xxx系列函数直接实现，不必大量的代码,但不支持sm2的签名，sm2的加解密
* 有一个sm3, sm4的比较好的代码： https://github.com/lizhichao/sm  可以使用composer安装，只是这个的ecb, cbc没有做补齐

### 使用(how to use)
* composer require lpilp/guomi 
* please make sure you upgrade to Composer 2+
* 测试是在php 7.4下做的，支持7.2及以上版本
### SM2
* 签名验签算法主体基于PHPECC算法架构，添加了sm2的椭圆参数， 
* 参考了 https://github.com/ToAnyWhere/phpsm2 童鞋的sm2验签算法，密钥生成算法
* 添加了签名算法， 支持sm2的16进制，base64公私钥的签名，验签算法
* 支持从文件中读取pem文件的签名，验签算法
* 添加了sm2的非对称加密的算法，但速度一般，有待优化，不能保证兼容所有语言进行加解密，目前测试了js, python的相互加解密
* sm2的加密解密算法在openssl 1.1.1的版本下自带的函数中暂无sm2的公钥私钥的加密函数，得自己实现，建议使用C，C++的算法，打包成PHP扩展的方式
* 由于 openssl没有实现sm2withsm3算法，用系统函数无法实现签名及证书的自签名分发

### SM3
* 该算法直接使用 https://github.com/ToAnyWhere/phpsm2 中sm2签名用到的匹配sm3, 未做修改
* 也可使用 openssl的函数, 详见openssl_tsm3.php

### SM4
* 该算法直接封装使用 https://github.com/lizhichao/sm  的sm4算法， 同时该项目支持 sm3,sm4 ,可以composer安装
* 由于sm4-ecb, sm4-cbc加密需要补齐，项目lizhichao/sm项目未做补齐操作，这里封装的时候，针对这两个算法做了补齐操作， 其他如sm4-ctr,sm4-cfb,sm4-ofb等，可以直接用
* 在openssl 1.1.1下可使用系统的函数，已支持sm4-cbc,sm4-cfb,sm4-ctr,sm4-ecb,sm4-ofb，  详见openssl_tsm4.php

### 总结
* 这里封装的测试函数已与相关的js, python, java都可以互签互认
* js: https://github.com/JuneAndGreen/sm-crypto
* python: https://github.com/duanhongyi/gmssl
* java: https://github.com/ZZMarquis/gmhelper
* openssl: 升到1.1.1以后，支持sm3,sm4的加解密，还不支持sm2的公私钥加解密，也不支持sm2的签名，得使用原生代码实现，签名中需要实现sm2withsm3, openssl1.1.1只实现了sm2whithsha256
