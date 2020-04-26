###你不知道的Nginx（一）

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge6udnevw3j30q40dot8u.jpg)

最近看到很多Nginx分享的文章，感觉有些配置自己理解的有些出入，重新梳理一下。

Nginx介绍、安装这里没有，直入主题--**gzip压缩配置**

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge64lml2itg306o06ojte.gif)



###开启压缩的两种姿势
####1、服务端压缩（动态）
仅配置**gzip: on;** 是不行的，需要指定压缩配置
这种压缩都是**动态的**，在**每次请求**都会先压缩再输出，大大**浪费**cpu。除非有代理缓存。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge64lqmi5ig306o06ojs9.gif)

```bash
# 开启压缩  
gzip  on;   
# 设置允许压缩的页面最小字节数，页面字节数从header头的content-length中进行获取。默认值是0，不管页面多大都压缩。建议设置成大于2k的字节数，小于2k可能会越压越大。  
gzip_min_length 2k;  
# 设置系统获取几个单位的缓存用于存储gzip的压缩结果数据流。 例如 4 4k 代表以4k为单位，按照原始数据大小以4k为单位的4倍申请内存。 4 8k 代表以8k为单位，按照原始数据大小以8k为单位的4倍申请内存。  
# 如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储gzip压缩结果。  
gzip_buffers 4 16k;  
#压缩级别，1-10，数字越大压缩的越好，也越占用CPU时间  
gzip_comp_level 2;  
# 默认值: gzip_types text/html (默认不对js/css文件进行压缩)  
# 压缩类型，匹配MIME类型进行压缩  
# 不能用通配符 text/*  
# (无论是否指定)text/html默认已经压缩   
# 设置哪种压缩文本文件可参考 conf/mime.types  
gzip_types text/plain application/x-javascript text/css application/xml;    
# 值为1.0和1.1 代表是否压缩http协议1.0，选择1.0则1.0和1.1都可以压缩  
gzip_http_version 1.0   
# IE6及以下禁止压缩  
gzip_disable "MSIE [1-6]\.";   
# 默认值：off  
# Nginx作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含"Via"的 header头。  
# off - 关闭所有的代理结果数据的压缩  
# expired - 启用压缩，如果header头中包含 "Expires" 头信息  
# no-cache - 启用压缩，如果header头中包含 "Cache-Control:no-cache" 头信息  
# no-store - 启用压缩，如果header头中包含 "Cache-Control:no-store" 头信息  
# private - 启用压缩，如果header头中包含 "Cache-Control:private" 头信息  
# no_last_modified - 启用压缩,如果header头中不包含 "Last-Modified" 头信息  
# no_etag - 启用压缩 ,如果header头中不包含 "ETag" 头信息  
# auth - 启用压缩 , 如果header头中包含 "Authorization" 头信息  
# any - 无条件启用压缩  
gzip_proxied expired no-cache no-store private auth;  
# 给CDN和代理服务器使用，针对相同url，可以根据头信息返回压缩和非压缩副本  
gzip_vary on;  
```


####2、前端构建时压缩（静态）

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge6ug2m3ipg306o06o0t4.gif)



```bash
# gzip_static是nginx对于静态文件的处理模块
# 该模块可以读取预先压缩的gz文件
# 这样可以减少每次请求进行gzip压缩的CPU资源消耗。
# 启用后，nginx首先检查是否存在请求静态文件的gz结尾的文件
# 如果有则直接返回该gz文件内容。
nginx_static on;
```

前端构建时使用**webpack插件** compression-webpack-plugin

```javascript
new CompressionWebpackPlugin({
  filename: '[path].gz[query]',
  algorithm: 'gzip',
  test: new RegExp('\\.(' + ['js', 'css'].join('|') + ')$'),
  // 只有大小大于该值的资源会被处理。单位是 bytes。默认值是 0
  threshold: 10240,
  // 只有压缩率小于这个值的资源才会被处理。默认值是 0.8
  minRatio: 0.8,
  cache: true
})
```



配置后打包，对应目录会生成**同名的gz文件** (这里选取2个文件为例)



![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge6ugoj631g306o06omxp.gif)

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge6uc4a27yj30p60fego8.jpg)



###是时候展示真正的技术了

#### 场景1 关闭gzip 请求原始文件

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5ugmdplfj311m0asada.jpg)



![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge64krkbwpg306o06omxj.gif)

#### 场景2 开启gzip 服务器动态压缩

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge6tx1jvxoj316y0he77s.jpg)

gzip_comp_level**设置为1、2** 效果对比

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5upjydzlj31200sw47m.jpg)



![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge64gp525gg306o06oq3j.gif)

![弱校验的ETag](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5uu1hu5kj30wg0em797.jpg)

#### 场景3 关闭gzip 开启静态模块

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge6txccwrkj317i0hegp3.jpg)

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5uvm369tj312008iacv.jpg)

![强校验的ETag](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5uyero5kj30uo08o0vo.jpg)



![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge64m7kn34g306o06o757.gif)

### ETag W/是干啥的

"5ea29704-47d8c"  -- 强校验的ETag

W/"5ea29704-f6764"  -- 弱校验的ETag

解释：

**强校验的ETag**匹配要求两个资源内容的**每个字节需完全相同**

**弱校验的ETag**匹配要求两个资源在**语义上相等**，这意味着在实际情况下它们可以互换，而且缓存副本也可以使用。不过这些资源**不需要每个字节相同**，因此弱ETag不适合字节范围请求。当Web服务器无法生成强ETag不切实际的时候，**比如动态生成的内容**，弱ETag就可能发挥作用了。

看下服务器上的**原始文件** 主要下**观察.gz文件大小**

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5v5ray7ij316u0de41o.jpg)

可以看出原始文件大小为29284字节，对应上图响应头content-length： **294284**

说明此时为强校验，gzip_static生效，使用的是构建时提前压缩的gz文件

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge6ul026rxg306o06odgt.gif)



### 压缩等级的选择 gzip_comp_level



如果使用**服务器压缩**，下图是压缩等级0-9 对应的**文件大小**，可以看出**值大于1时，压缩比例没有明显区别**，所有一般使用时**设为1或2**即可

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5trusq26j30s00kejw9.jpg)



![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge653hkiivg306o06owg9.gif)

### 参考资料

[1] https://www.cnblogs.com/zs-note/p/9556390.html

[2] https://www.iteye.com/blog/phl-2253442

[3] https://serverfault.com/questions/253074/what-is-the-best-nginx-compression-gzip-level

[4] https://www.cnblogs.com/Renyi-Fan/p/11047490.html

***
> 微信公众号：**[web程序员](#jump_10)**
> 关注可了解更多干货。问题或建议，请公众号留言;

![](https://tva1.sinaimg.cn/large/007S8ZIlly1ge5wijth53j31bi0hcwhg.jpg)