## 常用命令练习题

### dns
1. 使用dig命令获取www.baidu.com域名的解析记录；
2. 1中分别指定使用202.106.0.20和119.29.29.29这两个DNS解析www.baidu.com，查看结果，并且说明为什么存在差异？
3. 使用dig命令分析出www.baidu.com对应的ns服务器信息；
4. 如何优化提升DNS的效果；

### 文件处理
1. 在tmp目录下，新建一个以自己名字命令的txt文件，并且将文件权限修改为777；
2. 打开文件，加入以下内容，并且保存退出：
```
117.172.20.28 - - [30/Nov/2016:16:30:01 +0800] "HEAD http://v.meituan.net/mobile/app/Android/group-431_1-xiaomi.apk HTTP/1.1" 200 1 0 569 "-" Mozilla/5.0+(compatible;+pycurl)" "http://www.baidu.com" "-" "HIT" "117.156.21.143"  
117.169.75.234 - - [30/Nov/2016:16:30:00 +0800] "HEAD http://v.meituan.net/mobile/app/Android/group-431_1-xiaomi.apk HTTP/1.1" 200 0 0 563 "-"  Mozilla/5.0+(compatible;+pycurl)" "-" "-" "HIT" "112.29.134.74"
```

### 正则表达式
1. 过滤出文件处理环节保存的文件内容中的IP信息；
2. 过滤出对应的请求url信息；
3. 客户要求对www.baidu.com这个域名，针对jpg、png文件缓存30天，请写一个这个则用来匹配www.baidu.com域名的jpg和png文件；

### 网络探测
1. 使用mtr命令，探测到14.215.177.38这个IP的网络情况；每隔0.5秒发送一次探测，包大小设置为1500字节；

### curl命令
1. 请求http://res.cocounion.com//stuff/img/201611/353930438c1d6fc576b7a43ece840bbb.jpg ，文件保存为test.jpg，输出对应的请求头、响应头；
2. 对以上URL发起HEAD请求，获取响应头信息；
3. 向120.41.38.78这个服务器IP请求上述URL，并且请求头携带Usr: test头部信息；
4. 请求以上url的前1000个字节，并且观察响应状态码和之前的有什么区别；


### tcpdump抓包
在虚拟机上使用curl请求http://www.baidu.com/ 这个url，并且使用tcpdump对请求进行抓包，保存对应的抓包文件，使用wireshark查看请求头、响应头信息