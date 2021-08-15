# bilibiliupload
![](https://img.shields.io/badge/python-v3.7%2B-blue)
![GitHub](https://img.shields.io/github/license/ForgQi/bilibiliupload)

支持自动录制各大直播平台，上传直播录像到bilibili。  

* 自动选择上传线路，保证国内外vps上传质量和速度
* 可分别控制下载与上传并发量
* 支持Web API与客户端API上传

相关配置示例在config.yaml文件中，如直播间地址，b站账号密码

演示视频：[BV1ip4y1x7Gi](https://www.bilibili.com/video/BV1ip4y1x7Gi)
## INSTALLATION
1. 创建配置文件**config.yaml**，内容参照**config(demo).yaml**（或可调整配置后直接修改文件名）
2. 安装 __FFmpeg__, __pip__
3. 安装 __biliup__：
`pip3 install biliup`
```shell
# 启动
$ biliup start
# 退出 
$ biliup stop
# 重启 
$ biliup restart
# 查看版本
$ biliup --version
# 显示帮助
$ biliup -h
```

Linux下以daemon进程启动，录像和日志文件保存在执行目录下，程序执行过程可查看日志文件。
`ps -A | grep biliup` 查看进程是否启动成功。


## Docker使用 🔨 
### 方式一
```bash
vim /host/path/config.yaml
docker run --name biliup -v /host/path:/opt -d ghcr.io/forgqi/bilibiliupload/caution
```
### 方式二
```bash
cd bilibiliupload
sudo docker build . -t sc2
sudo docker run -d sc2
```
### 进入容器 📦
```bash
sudo docker ps (找到你的imageId)
sudo docker exec -it imageId /bin/bash     
```

## 源 码 使 用
* 下载源码: git clone https://github.com/ForgQi/bilibiliupload.git
* 安装: `pip3 install -e .` 或者 `pip3 install -r requirements.txt`
* 启动: `python3 -m biliup`

## EMBEDDING BILIUP
如果你不想使用完全自动托管的功能，而仅仅只是想嵌入biliup作为一个库来使用这里有两个例子可以作为参考
### 上传
```python
from biliup.plugins.bili_webup import BiliBili, Data

video = Data()
video.title = '视频标题'
video.desc = '视频简介'
video.source = '添加转载地址说明'
# 设置视频分区,默认为160 生活分区
video.tid = 171
video.set_tag(['星际争霸2', '电子竞技'])
with BiliBili(video) as bili:
    bili.login_by_password("username", "password")
    for file in file_list:
        video_part = bili.upload_file(file)  # 上传视频
        video.videos.append(video_part)  # 添加已经上传的视频
    video.cover = bili.cover_up('/cover_path').replace('http:', '')
    ret = bili.submit()  # 提交视频
```
### 下载
```python
from biliup.downloader import download

download('文件名', 'https://www.panda.tv/1150595', suffix='flv')
```
## 使用建议
关于B站为什么不能多p上传\
目前bilibili网页端是根据用户权重来限制分p数量的，权重不够的用户自动切换到客户端的提交接口。
>用户等级大于3，且粉丝数>100，web端投稿不限制分p数量

国内VPS网络费用较高，建议使用国外VPS，根据机器的硬盘等资源设置合理并发量。

b站上传目前有两种模式，分别为bup和bupfetch模式。
>* bup：国内常用模式，视频直接上传到b站投稿系统。
>* bupfetch：目前见于国外网络环境，视频首先上传至第三方文件系统，上传结束后通知bilibili投稿系统，再由b站投稿系统从第三方系统拉取视频，以保证某些地区用户的上传体验。

bup模式支持的上传方式为upos，其线路有：
* ws（网宿）
* qn（七牛）
* bda2（百度）

bupfetch模式支持的上传方式及线路有：
1. kodo（七牛）
2. gcs（谷歌）
3. bos（百度）

国内基本选择upos模式的bda2线路。国外多为upos模式的ws和qn线路，也有bupfetch模式的kodo、gcs线路。bilibili采用客户端和服务器端线路探测相结合的方式，服务器会返回可选线路，客户端上传前会先发包测试选择一条延迟最低的线路，保证各个地区的上传质量。
***
登录有两种方案：

* 操作浏览器模拟登录

* 通过b站的OAuth2接口

>对于滑动验证码可进行二值化、灰度处理找缺口计算移动像素，系统会上传分析你的拖动行为，模拟人操作轨迹，提供加速度、抖动等，如直接拖动到目标位置不能通过验证，提示：“拼图被怪物吃了”。滑动验证码系统会学习，需不断更新轨迹策略保证通过验证的成功率。\
>OAuth2接口要提供key，需逆向分析各端

线程池限制并发数，减少磁盘占满的可能性。检测下载情况卡死或者下载超时，重试三次保证可用性。代码更新后将在空闲时自动重启。


下载整合了ykdl、youtube-dl、streamlink，不支持或者支持的不够好的网站可自行拓展。
下载和上传模块插件化，如果有上传或下载目前不支持平台的需求便于拓展。

下载基类在`engine/plugins/base_adapter.py`中，拓展其他网站，需要继承下载模块的基类，加装饰器`@Plugin.download`。

拓展上传平台，继承`engine/plugins/upload/__init__.py`文件中上传基类，加装饰器`@Plugin.upload`。

实现了一套基于装饰器的事件驱动框架。增加其他功能监听对应事件即可，比如下载后转码：
```python
# e.p.给函数注册事件
# 如果操作耗时请指定block=True, 否则会卡住事件循环
@event_manager.register("download_finish", block=True)
def transcoding(data):
    pass
```

## Deprecated
* ~~selenium操作浏览器上传两种方式~~(详见bili_chromeup.py)
* ~~Windows图形界面版在release中下载AutoTool.msi进行安装~~[AutoTool.msi](https://github.com/ForgQi/bilibiliupload/releases/tag/v0.1.0)
QQ群：837362626
## Credits
* Thanks `ykdl, youtube-dl, streamlink` provides downloader.

类似项目:\
![ZhangMingZhao1](https://avatars2.githubusercontent.com/u/29058747?s=50&u=5f8c3acaa9d09f4396f00256c0ce6ef01452e92f&v=4) ：StreamerHelper
