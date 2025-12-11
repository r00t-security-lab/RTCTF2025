## 导航的时空奶龙防御系统

题目非常简单，就是告诉了你管理员用户名是admin，让你登录，登陆成功就会有flag，同时提供了密码本，按道理来说非常基础简单。但是在我们抓包重放去爆破密码的时候发现出现了问题，重放的数据包并不能得到正常的响应。

观察发送的包发现除了username和password以外，还有一个random参数同时被上传了，random很明显是一个进行加密过后的内容，因为题目并未提供源代码，所以判断加密逻辑大概率写在前端，ctrl+u看一下

 <img src="assests\image-20251209143417944.png" alt="image-20251209143417944" style="zoom:50%;" />

发现前端写着一个加密逻辑，同时把公钥提供给了我们，简单看看加密逻辑，发现就是获取当前的时间戳，对他进行aes加密之后作为参数提交，服务端会解密看时间戳是否符合当前时间，不符合就no repeat，那此时方法就很清晰了，可以选择使用python进行发包，因为python更方便编写加密逻辑，这里我们再提供另一种方式，就是使用yakit的热加载功能

<img src="assests\image-20251209144303244.png" alt="image-20251209144303244" style="zoom:50%;" />

实际上就是可以编写一个函数，在发送数据包的过程中可以调用这个函数进行实时地无感知的计算，那这里我们只需要写一个同样的加密函数，让random=这个函数返回的结果就可以了

```yaklang
var GlobalPubKey = ""

// 修改参数名为 unusedArg，避免冲突
calc_random = func(unusedArg) {
    if GlobalPubKey == "" {
        targetUrl = "http://127.0.0.1:5000/"
        rsp, _, err = poc.Get(targetUrl)
        
        if err != nil { return "ERROR_GETTING_PAGE" }

        bodyBytes = rsp.GetBody()
        pattern = `const PUBLIC_KEY = \x60(?s:(.*?))\x60`
        matches = re.FindSubmatch(bodyBytes, pattern)
        
        if len(matches) < 2 { return "ERROR_NO_KEY_FOUND" }
        
        GlobalPubKey = matches[1]
    }

    tsStr = sprint(time.Now().Unix())
    
    result, err = codec.RSAEncryptWithPKCS1v15(GlobalPubKey, []byte(tsStr))
    if err != nil { return "ERROR_ENCRYPT" }

    b64 = codec.EncodeBase64(result)
    return codec.EscapeQueryUrl(b64)
}
```

这里我们直接从源码里读公钥，因为每次环境更新的话密钥会不同，直接从代码中获取即可，模板使用`{{yak(calc_random|{{randint(1,10000000)}})}}`

这里用randint的原因是，你每个包都需要读取一下当前时间，都需要重新进行一次加密，所以让他参数不同，不然他只会执行第一次

然后发包筛选响应大小找到即可

<img src="assests\image-20251209144746828.png" alt="image-20251209144746828" style="zoom:50%;" />