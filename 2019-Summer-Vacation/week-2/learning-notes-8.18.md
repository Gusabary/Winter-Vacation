# Learning notes of Week 2

## 8.18 Sun.

+ >Shell也被称为“壳”或“壳程序”，它是用户与操作系统内核交流的翻译官，简单的说就是人与计算机交互的界面和接口。目前很多Linux系统默认的Shell都是bash（Bourne Again SHell），因为它可以使用tab键进行命令和路径补全、可以保存历史命令、可以方便的配置环境变量以及执行批处理操作。

+ `ls -al` 列出的详细信息为文件类型、所有者权限、同组用户权限、其他用户权限、拥有者、拥有者所在组。

+ > **反向代理**在计算机网络中是代理服务器的一种。服务器根据客户端的请求，从其关系的一组或多组后端服务器上获取资源，然后再将这些资源返回给客户端，客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器集群的存在。


### Django

+ 在虚拟环境中使用 `pip install -U pip` 会报无法将文件移到不同磁盘的错误，然后 `pip install` 也下载不了东西，但是使用 `python -m pip install -U pip` 就没有问题，再用 `pip install` 也能下载东西了。

  可能的原因是创建虚拟环境时一开始是没有 pip 的，如果直接用 `pip install` 会去用 C 盘的，即使可以通过 `python -m ensurepip` 安装 pip，安装的也是 9.0 版本的，下载东西还是会有问题。所以要用 `python -m pip install -U pip` 装 19.2 版本的 pip，然后才能正常使用。

+ > 一个Django项目可以包含一个或多个应用。

##### Last-modified date: 2019.8.18, 10 p.m.
