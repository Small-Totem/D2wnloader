# D2wnloader

```python
#Small_Totem:在ipynb脚本中使用
!git clone https://github.com/Small-Totem/D2wnloader.git
import D2wnloader.D2wnloader as d2l
url = "..."
dl = d2l.D2wnloader(url,blocks_num=64)
dl.start()
```

**Python** **多线程下载** **分块下载** **断点续传**

由于手机尤其是 iPhone 根本没有下载工具，官方不允许这种东西上架。

因此考虑用 Python 写了这个下载器，当然，想在 iPhone 上跑脚本你需要先安装 Pythonista 或者类似的东西。  

开源的 a-shell 很好用的，如果用 shortcuts 调用时需要点小技巧：先打开应用，等待 1 秒，再执行你的 py 程序，给点准备时间。  

应用程序 play.js 在 iOS 上似乎也不错。

## D1wnloader

也就是原来 UTOPIA-Downloader 中的 main.py  

## D2wnloader

更新原因是增加了以下需求：

- 经常越下越慢，希望速度下降时能自动重建连接重振雄风
- 需要让已完成任务的进程能够去分担还没完成的部分
- 需要一个可随时改变的分块算法

### 设计思路

AAEK 据说是某五笔输入法【未开垦】的编码，无从考证。格式为 `AAEK = [(0,4), (5,11), (12,99) ...]`，用此形式表示未下载的部分，每对 tuple 表示 (开始,结束)，两头包含。所以：

- 如果 100 个字节的文件尚未下载时为：`[(0,99)]`，当下载了 50 至 58 字节后变成 `[(0,49),(59,99)]`。
- 根据 AAEK 可以推算出已下载了那些字节。
- 缓存文件命名：特征值_起始字节，比如 MD5XXX.0, MD5XXX.59。也同样需要合并机制，合并过程需要包含核对文件大小，也需要核对是否与 AAEK 矛盾。
- 分块小于一个阈值就可以不再分割了。

工作流程：

- 获取要下载文件的大小；
- 启动测速线程；当发现速度明显下降，重置所有连接；
- __ask_for_work 申请与 block_num 数量相当的下载分块：
    1. 充足就返回；
    2. 不够就尝试分割；
    3. 没有分块，先返回 []，然后尝试调用一个 worker 的 help；
- 跑起来的 worker 有 3 种结局：
    1. 下载完毕，要求 __ask_for_work。如果还有任务，转生另 1 个 worker 继续下载；
    2. 请求帮助。自己提前结束，未完成的部分交出，申请另 2 个 worker 继续下载。相当于线程 +1；
    3. 退休。仅交出未完成的部分，不在继续。用于希望减少线程数；
- 下载结束，组装缓存文件，计算 md5 值；

### 用法

``` python
d2l = D2wnloader("https://qd.myapp.com/myapp/qqteam/pcqq/QQ9.0.8_3.exe")
d2l.start()
```

### 补充说明
我对 http 协议还在间断而不系统地学习中；我也不太会用 GitHub 忘见谅。
近期在降低 Docker 内存占用研究时发现了 D2w 的几个问题，现已修复：

- restart 会“卡住”没有接盘侠；
- 服务器给出的 5xx 错误也会被写入文件；

树莓派 Zero 搭建下载测试环境（等于服务器性能很差）下载全是 \x00 的文件时发现之前并未检查 status code
