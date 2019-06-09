---
title: 分析网易云的JS加密
date: 2019-05-24 12:15:27
tags: [爬虫,Python,javascript]
categories: 爬虫
---
### 起源
最近想搞一个工具App，需要网易云的数据，所以要分析网易云的WebAPI。之前也写过很多爬虫，所以刚开始我是信心满满的，直到看见网易云的请求都是这个样子的：
```
params: wVJpucPzzULoYCnoTXT809yc/SIgz8dCFVc6YBqaUhNzJ2YF2y5uA/vfByF1bQbLwI/5c9iIa3aHaEN2OMe+siNTh+MLWS6nqJKpn0P28SJ1BlO5uE7t8UPte8uOb3AoSLMtHkZ3Wqp8HmAhpVuc+u5xMoECk19oLTDGxYMjeBB1Y19gLKszcHi+thmTIn8N2BvR2EDVfS2CWWUvQsD2msJghlh9/eglF8uz4aKPA8NeSXccF8HxieefeWsJ6//jk0wYQMAaxeDqIJQiekFFMw==
encSecKey: d91f190d99d06d2400e0817cfbb7abfae5de48ea26091b42cb62eef0379350babd87078dfd3c9ac36eeef02e2e46dc9335e965bf2a9b3c99773caf31003dd4e0a5cdc710e2e44a26355231a44d7647d92b92d68c2e3a84951d36cedf92139241a8444a874b6b175f16c7401fea31770b0ebc6d2f09b444dd5807f38db1f53873
```
我就崩溃了，这肯定都是通过JS加密的，以前应对JS加密我从来都是用Selenium这种库+无头浏览器，效率底下还占资源，关键是应用场景有限，只是爬取数据用它可以，但是我要搞一个App的话，用Selenium就不行了。

于是我只能分析它的JS代码，但是一般看见的js为了加载速度都是经过压缩的，网易云当然也是，几万行在一个文件里，变量名都是abcdefg，用开发者工具弄可能不太方便，效率也不高，而且你想改js代码应该是实现不了\(虽说可以打断点\)。于是百度了一下，找到了一个思路，就是用代理拦截对应的js请求，替换成本地的，这样就可以在本地的编辑器里编辑和阅读JS了。

于是。。因为一点经验都没有，我搞了一上午，不过还是搞出来了，哈哈哈瞬间超级开心。

### 前戏
用到以下工具：
```
Charles(Windows下用Fiddler也可以 或者可以用Mono Fiddler)
一个浏览器(Chrome Dev)
一个编辑器(VSCode)
```
打开Charles，点`Help->SSL Proxying->Save Charles Root Certificate`保存Charles的根证书，因为我们要抓取的东西是HTTPS的，所以要用到。

然后保存的证书是cer的，我们要转成crt并且复制到`/usr/share/ca-certificates`，并安装证书。
```bash
openssl x509 -inform der -in CharlesRoot.cer -outform pem -out CharlesRoot.crt
sudo cp CharlesRoot.crt /usr/share/ca-certificates
sudo dpkg-reconfigure ca-certificates //选择ask,勾选CharlesRoot.crt并确认
```
然后点击`Proxy->Proxy Setting`，标签Proxies下勾选Enable transparent HTTP proxying。

设置你的系统或浏览器使用你的代理端口，然后好像还得把你的证书安装到浏览器里，反正我的Chrome安装了。之后你就能看到浏览器收发的所有数据包了，你点开一个URL之后会发现全都是`Unknow`，这是因为你没开启该URL的HTTPS代理，右键该URL之后选择`Enable SSL Proxying`。

现在应该就能看到该URL下HTTPS的流量了。
### 开始分析
PS：因为分析的目标JS代码经过压缩，基本不是给人看的，很恶心，所以注意力要集中哦。。。

我们就用搜索单曲的例子来说吧，首先打开网易云音乐，然后搜索，在开发者工具的XHR请求里你应该会看到这个URL，这就是搜索的URL。

![IMG](http://prgs9av8q.bkt.clouddn.com/jt.png)

然后看到这是一个POST请求，直接去看它的Form Data，看到两串BASE64编码的奇怪的参数，一个是`params`一个是`encSecKey`，尝试直接Base64解密，解密后还是一堆加密过的参数，于是我们去分析它的JS。

![IMG](http://prgs9av8q.bkt.clouddn.com/jt2.png)

我们搜索一下关键词，encSecKey，发现只有一个叫`core_`什么的js文件中有。

![IMG](http://prgs9av8q.bkt.clouddn.com/jt3.png)

点进去，麻痹，45792行。。。。直接复制到VSCode里，保存到一个本地文件中，之后我们就分析这个文件。然后让我们的Charles去拦截并替换它。

点击`Tools->Map Local`然后Enable Map Local，之后点击Add去添加一个文件映射。

![IMG](http://prgs9av8q.bkt.clouddn.com/jt4.png)

Host填那个JS文件的所在的主机，Path填JS文件名，这里要注意把后面的参数去了，Query选项才是填参数的地方，不过这里我们不管它参数是啥都拦截，所以就填星号。在Local Path中选择你本地文件所在的位置。

完事，我们在第一行加一个`alert`或者注释，看一下是否真正替换了。

替换完之后我们开始分析JS，先看哪个参数？先看长的，`encSecKey`这个，我们在整个JS文件里搜索它。

发现一共就三处，最后两处在同一个地方，并且在那还发现了params，所以我们从这开始分析。

```javascript
var bUP3x = window.asrsea(JSON.stringify(i4m), brC5H(["流泪", "强"]), brC5H(WU0x.md), brC5H(["爱心", "女孩", "惊恐", "大笑"]));
e4i.data = k4o.cz5E({
    params: bUP3x.encText,
    encSecKey: bUP3x.encSecKey
})
```
基本可以肯定，这里就是我们form data里看到的参数，这两个参数都来自于`bUP3x`，而这个变量是调用了`window.asrsea`的返回值，所以我们应该去asrsea中找，搜索一下会找到如下一段代码：
```javascript
!function() {
    //...省略一些代码
    window.asrsea = d,
    window.ecnonasr = e
}();
```
可以看到这里把d作为了`window.asrsea`的值，那么d是啥？d其实就在省略的代码里面，这里面有五个函数，分别是a b c d e，我们首先着重看d。
```javascript
function d(d, e, f, g) {
    var h = {}
        , i = a(16);
    return h.encText = b(d, g),
    h.encText = b(h.encText, i),
    h.encSecKey = c(i, e, f),
    h
}
```
d有4个参数，是刚才我们分析的`bUP3x`那里传进去的，我们先不考虑它是啥，等分析完这段在讨论。我们先把它当成已知的。

可以看到这里对于encText和encSecKey都赋值了，根据之前的代码我们可以知道，encText就是formdata里的params，encSecKey就是encSecKey。所以这个方法就是加密的核心。

它对encText做了两次赋值，都是调用b，我们也可以看做`b(b(d,g),i)`，我个人感觉这样看比分开看清晰，因为这样看它们之间的关系比较强。

我们看看b的代码。

```javascript
function b(a, b) {
    var c = CryptoJS.enc.Utf8.parse(b)
        , d = CryptoJS.enc.Utf8.parse("0102030405060708")
        , e = CryptoJS.enc.Utf8.parse(a)
        , f = CryptoJS.AES.encrypt(e, c, {
        iv: d,
        mode: CryptoJS.mode.CBC
    });
    return f.toString()
}
```

b其实比较简单了，不过经过代码压缩之后看起来很乱，我们可以在脑海中给它修改一下，不难看出就是把第二个参数作为key，把一个字符串`0102030405060708`作为偏移量，然后对第一个参数进行AES加密，模式是CBC。

我们再看一下d中对b的调用，是这样的`b(b(d,g),i)`，也就是说先用参数g作为key对参数d进行一次aes加密，再把i作为key和结果进行aes加密，d和g是外面传过来的，那i是哪来的？

d中有这样一行
```javascript
var h = {}
    , i = a(16);
```
i是调用了a之后来的，我们继续分析a函数。
```javascript
function a(a) {
    var d, e, b = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789", c = "";
    for (d = 0; a > d; d += 1)
        e = Math.random() * b.length,
        e = Math.floor(e),
        c += b.charAt(e);
    return c
}
```
一眼就能看出这是生成了一个长度为a的随机字符串，而a即d中传过来的参数16，也就是说函数a返回长度为16的随机字符串。

现在我们就知道encText是怎么来的了，我们知道params就是encText，所以我们已经解开了一个参数的加密过程了，但是还有一个疑问，就是d方法的参数都是啥，我们还不知道。我们回到调用`window.asrsea`的地方\(因为通过分析我们知道d就是window.asrsea\)去看看。

```javascript
var bUP3x = window.asrsea(JSON.stringify(i4m), brC5H(["流泪", "强"]), brC5H(WU0x.md), brC5H(["爱心", "女孩", "惊恐", "大笑"]));
e4i.data = k4o.cz5E({
    params: bUP3x.encText,
    encSecKey: bUP3x.encSecKey
})
```
还是这段代码，我们发现第一个参数是一个由i4m变的json字符串，第二个第四个应该都是写死的，我也不知道为啥是这些，我也没分析，它都写死了，我们也不用浪费时间到这个上面。

我们再看看第三个参数，是`WU0x.md`，找到这个变量的定义，其实也是一个写死的字符串，而且很长，我们不难看出第2，3，4个变量都是一些emoji的名字，不过为啥要加，我们无从得知，我们也不需要得知，既然它写死了，我们就直接复制，写爬虫就是这样嘛，能走捷径就走捷径。

我们加两个console.log，把经过这些emoji经过`brC5H`之后的值打印出来。顺便打印出第一个参数。

```javascript
console.log(JSON.stringify(i4m));
console.log(brC5H(["流泪", "强"]), brC5H(WU0x.md), brC5H(["爱心", "女孩", "惊恐", "大笑"]));
```
刷新下网页，重新请求，然后看控制台的输出，一次搜索操作`window.asrsea`方法被调用了多次，第一个参数是改变的，剩下三个参数的值是恒定不变的，而且我们可以看出`brC5H`方法是个加密方法或者是编码方法，不过因为它值被写死了，导致每次加密的值都一样，我们也就不用分析这个方法了。

![IMG](http://prgs9av8q.bkt.clouddn.com/jt5.png)

既然调用了多次，我们要知道哪次调用返回的结果才是我们发出请求的formdata中的参数，于是继续添加log，看看每次`window.asrsea`的返回值。


```javascript
console.log(bUP3x);
```
window.asrsea返回值   
![window.asrsea返回值](http://prgs9av8q.bkt.clouddn.com/jt6.png)

formdata参数  
![formdata参数](http://prgs9av8q.bkt.clouddn.com/jt7.png)

我们可以找到这一次的返回值和参数对应上了，于是我们就看一看这次调用时它的可变参数，也就是第一个参数是什么。
```json
{"hlpretag":"<span class=\"s-fc7\">","hlposttag":"</span>","s":"asdfasd","type":"1","offset":"0","total":"true","limit":"30","csrf_token":""}
```
一个json字符串，没经过格式化的，我们立马可以看出它就是我们的搜索参数，s即搜索关键字\(我瞎打的\)，offset即偏移位置，limit返回多少个。

那么我们原封不懂的把他copy下来备用，然后别忘了，encSecKey这个参数我们还没有研究呢。

继续回到函数d。

```javascript
function d(d, e, f, g) {
    var h = {}
        , i = a(16);
    return h.encText = b(d, g),
    h.encText = b(h.encText, i),
    h.encSecKey = c(i, e, f),
    h
}
```
可以看到encSecKey调用了函数c，并且把i和ef传了进去，我们知道i是随机变量，e和f是写死的，那么我们不妨做出大胆的猜想，我们每次把随机字符串写死那么encSecKey应该返回的是同一个东西。

这样做的话，参数e和f我们不用管了，也就是说我们关心的就是d和g，d我们知道了是那个json字符串，g的话我们去之前打印的结果里复制下就行了。efg这三个参数是不会变的。

```
//这是g
0CoJUm6Qyw8W8jud
```

而且encText中也用到了i，写死的话，我们生成16位随机字符串的算法也不用写了。

理一理头绪开始编码。
```
//准备一下这个json串，因为我们构造搜索关键字和返回结果时都用它控制
{"hlpretag":"<span class=\"s-fc7\">","hlposttag":"</span>","s":"$searchKeyWord","type":"1","offset":"0","total":"true","limit":"30","csrf_token":""}

//把上面的$searchKeyWord替换成想搜索的就行了

//我们知道params=b(b(d,g),i)
//那么先把这个json串以0CoJUm6Qyw8W8jud作为key进行aes加密，再把结果以随机变量i进行aes加密即可得到params

//encSecKey我们准备写死，所以直接copy下来就可以
//因为这个encSecKey完全是由随机变量i决定的，那么我们就把当时的i也复制下来
//因为之前的时候没打印i，所以还得加行代码并且刷新
//这下formdata中的两个参数都出来了，我们开始编码

//最后我们使用的encSecKey和随机变量如下
encSecKey： 0c76e2da719b4d3ab2ec44ccdd5100266f4b34cf6d7cd09c4de472aea01de6b1d476918225680680ddee41e02f1a5f19feee10bdea88c0b5bd7661ce5ab06212c7c53cce91d15ff6bc85e995251bb0ad6e240c9f2d4ae3d0631facc1dae8abc24703a66be6b98cd3250e6da7973e0286f7b6e3c1b474c8356961fc2126dc9ea7

i： pIuxxPXdZv2yDPbr
```

### 代码
因为我考虑弄的是Android的app，但是由于测试，先用Python实现了一下


```python
# 主文件
import requests as req
import netease_crypto
import json

search_str = "Midsummer Madness"

query_body = '{"hlpretag":"<span class=\\"s-fc7\\">","hlposttag":"</span>","s":"'+search_str+'","type":"1","offset":"0","total":"true","limit":"30","csrf_token":""}'
url = "https://music.163.com/weapi/cloudsearch/get/web?csrf_token="
params = netease_crypto.d(query_body)
data = {
    "params":params,
    "encSecKey":"0c76e2da719b4d3ab2ec44ccdd5100266f4b34cf6d7cd09c4de472aea01de6b1d476918225680680ddee41e02f1a5f19feee10bdea88c0b5bd7661ce5ab06212c7c53cce91d15ff6bc85e995251bb0ad6e240c9f2d4ae3d0631facc1dae8abc24703a66be6b98cd3250e6da7973e0286f7b6e3c1b474c8356961fc2126dc9ea7"
}
resp = req.post(url=url,data=data)
jo = json.loads(resp.text)
for song in jo['result']['songs']:
    print("歌曲："+song['name'],end='   艺术家：')
    for artist in song['ar']:
        print(artist['name'],end=' / ')
    print()
```
```python
# netease_crypto
import base64
from Crypto.Cipher import AES

def aes_en(key,msg):
    cryptor = AES.new(key,AES.MODE_CBC,b'0102030405060708')
    num = (16-len(msg)%16)
    msg = msg + (chr(num)*num)
    ret = cryptor.encrypt(msg)
    return base64.b64encode(ret).decode()
def d(d):
    ret_1 = aes_en('0CoJUm6Qyw8W8jud',d)
    print(ret_1)
    ret_2 = aes_en('pIuxxPXdZv2yDPbr',ret_1)
    return ret_2
```

结果：
```
歌曲：Midsummer Madness   艺术家：88rising / Higher Brothers / Rich Brian / Joji / AUGUST 08 / 
歌曲：Midsummer Madness(Howie X Remix)   艺术家：孔昊龑Howie X / 88rising / Joji / Higher Brothers / Rich Brian / AUGUST 08 / 
歌曲：Midsummer Madness   艺术家：88rising / Higher Brothers / Rich Brian / Joji / AUGUST 08 / 
歌曲：Midsummer Madness (KRANE Remix)   艺术家：88rising / KRANE / 
歌曲：Midsummer Madness (Cover)   艺术家：Kid Travis / 
歌曲：88rising-Midsummer Madness（MISS BLACK TIGER / Lil Sen / TXU徐泽一 remix）   艺术家：MISS BLACK TIGER / Lil Sen / TXU徐泽一 / 
歌曲：Midsummer Madness (19KILLS REMIX)   艺术家：19kills / 88rising / 
歌曲：Midsummer Madness (Drop-Zone Remix)   艺术家：Mr.Tac / 
歌曲：88rising-Midsummer Madness（BEAUZ remix）   艺术家：BEAUZ / 
歌曲：Midsummer Madness   艺术家：Instrumental Soon / 
歌曲：Midsummer Madness   艺术家：Ivor Slaney & His Orchestra / Dolores Ventura / Ivor Slaney / 
歌曲：Midsummer Madness   艺术家：Kid Creole & the Coconuts / 
歌曲：Midsummer Madness (Original Mix)   艺术家：Martin DP / 
歌曲：Midsummer Madness (Original Mix)   艺术家：Martin DP / 
歌曲：Midsummer Madness（Cover：Rich Brian）   艺术家：牟敏雪MichelleBla1r / 
歌曲：Midsummer Madness   艺术家：Sherbet / 
歌曲：Midsummer Madness（Cover：88rising/joji/Rich Brian/HigherBrothers/AUGUS）   艺术家：金菊 / 
歌曲：Midsummer Madness   艺术家：Ivor Slaney & His Orchestra / 
歌曲：Midsummer Madness   艺术家：Orchid_J / 
歌曲：Midsummer Madness （I N C I D E N T A L）   艺术家：MISERY / 
歌曲：Higher Brothers-Midsummer Madness (Remix)（P:ERSIIA / Monster-Jun / MISS BLACK TIGER remix）   艺术家：P:ERSIIA / Monster-Jun / MISS BLACK TIGER / 
歌曲：88rising-Midsummer Madness（叵测Cc Remix）   艺术家：叵测Cc / 
歌曲：88rising-midsummer madness（叵测Cc Remix）   艺术家：叵测Cc / 
...省略
```
### 值得注意的地方
别看我说的这么顺，其实实际我自己分析的时候没那么顺，我弄了一上午呢，把坑记录一下，下次注意。

如果对AES加密本身就很了解的可以忽略。

首先网易云的aes加密代码使用`CryptoJS`，我是没找到它的文档。。。我也不知道这是个啥库，因为我没做过JS开发。

由于AES加密的CBC算法需要加密字符串长度是16的倍数，所以就涉及到补齐问题，我寻思补齐应该是补'\0'，看各路大佬们写的代码也都是'\0'，但是`CryptoJS`默认的补齐方法采用一个叫`pkcs7`的补齐标准，所以不是补'\0'，而是补`16-len(text)%16`对应在ascii表中的字符，所以应该这样写。
```python
num = (16-len(text)%16)
text = text + (chr(text)*num)
```
### 参考
[ubuntu18.04中charles安装及使用-huuinn](https://blog.csdn.net/huuinn/article/details/82762952)