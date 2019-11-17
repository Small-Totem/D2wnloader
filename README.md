# UTOPIA-Downloader
**Python** **多线程下载** **分块下载** **断点续传**

由于手机尤其是 iPhone 根本没有下载工具，官方不允许这种东西上架。

因此考虑用 Python 写了这个下载器，当然，想在 iPhone 上跑脚本你需要先安装 Pythonista 或者类似的东西。

主要解决断点续传、多线程分块下载问题。

2019-11-17 更新：  
- 拼装前检查分块大小，如某分块未完成就结束则重试  
- 修正不重要的文案  


可能出现的 bug ：

* 如果网站并不返回 Content-Length 会引起错误，对于这种网站分块下载和断点续传也没什么意义；

* ~~如果第一次下载到一半不成功，第二次不能改 blocks_num 参数，否则就分块错乱了；~~ 已修正；

* 在 iPhone 7 上，目前发现下载超过1.7G的文件时，最后一步计算 sha256 会出现 Memory Error，但是那时已经“缝合”完毕，文件是完整的。当然也可以通过注释掉这个功能避免看见错误，不过这应该是 Pythonista 的问题；


