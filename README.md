# test
##project viewers
>me  

>you  
>they

## project going
* project1
	* project2
	* projet3

### 需求确认
**1.缓存**<br>
所有频道共用缓存，http和https共用缓存，缓存时间尽可能长;  
**2.回源**   
提供了电信、联通及移动3条回源链路。默认走电信、联通走联通、移动和铁通走移动   
**3. 其他特殊配置**
* 404 缓存5分钟
* 0字节文件不缓存
* 需要对文件md5做校验
