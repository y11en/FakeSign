# 自建时间戳服务器实现伪签名驱动证书
## Implementing Pseudo Signature with Self-Sign Timestamp Servers
## 免责声明 / Disclaimers
<details>
<summary>[Disclaimers - English]</summary>
This article involves network security experiments. Reading this article indicates that you have read, fully understand, and promise to comply with all the following terms and conditions:
> 1. You promise that the technology involved in this article will only be used for experimentation and security technology testing, and shall not be used for any criminal activities, fraud or cracking, nor for production environments that require confidentiality or importance.
> 2. You comply with the Cybersecurity Law of the People's Republic of China and are not allowed to use any technology on this website for illegal or criminal activities.
> 3. You shall comply with Article 286 (1) of the Criminal Law and shall not use any technology on this website to disrupt computer information systems.
> 4. You shall comply with Article 32 of the Electronic Signature Law and shall not use any technology of this website to forge, impersonate, or embezzle the electronic signature of others
> If any of the above terms are violated, we will fully and independently assume any legal and other responsibilities that may arise.
</details>
**本文涉及网络安全实验，阅读本文表示您已经阅读、完全理解并承诺遵守下列条款的全部内容**：

> 1、您承诺本文涉及的技术只被用于实验和安全技术测试，不得用于任何违反犯罪活动，不得用于欺诈或破解，不用于需要保密或者重要的生产环境。
> 2、您遵守《中华人民共和国网络安全法》，不得使用本网站任何技术进行违法犯罪活动。
> 3、您遵守《刑法》第286条第1款规定，不得使用本网站的任何技术破坏计算机信息系统。
> 4、您遵守《电子签名法》第32条，不得使用本网站的任何技术伪造、冒用、盗用他人的电子签名
> 5、您遵守中国以及其他所在国家和地区的法律法规，不得使用本网站的任何技术违反法律法规，或者给其他任何个人、团体造成问题或者损失。
> **如果违反上述条款的任何内容，将完全独立承担带来的任何法律以及其他责任。**

## 实现原理 / Principle



## 快速使用 / QuickUse

### CA证书：用于时间戳认证 / CA Certificate:  Used for Timestamp Auth

二选一，***需要和下面签名工具的时间戳证书一致***

- #### Pikachu Fake CA （推荐）：[自动安装工具（推荐）](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/pika-fake-root-cert.exe)

- #### JemmyLoveJenny（备用）：[注册安装工具（手动）](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/JemmyLoveJenny-cert.reg)

### 泄漏的过期签名代码证书 / Leaked Expired Signature Code Certificate

> ***需要2015-07-29及以前的EV代码签名证书***，**我不提供任何代码签名证书**
>
> ***EV code signing certificates from July 29, 2015 and earlier are required***,
>
> **I do NOT provide any code signing certificates**

### 亚洲诚信数字签名工具包 / TrustAsia Digital Signature Toolkit Modify

> #### 下载工具 / Download Tools
>
> 二选一，***需要和之前安装的CA证书一致***
>
> - ##### Pikachu Fake CA （推荐）：[亚洲诚信签名工具 / TrustAsia SignTool - PikaFakeTimers](https://github.com/PIKACHUIM/FakeSign/raw/main/SignTool/Released/HookSigntool-PikaFakeTimers.zip)
>
> - ##### JemmyLoveJenny（备用）：[亚洲诚信签名工具 / TrustAsia SignTool - JemmyLoveJenny](https://github.com/PIKACHUIM/FakeSign/raw/main/SignTool/Released/HookSigntool-JemmyLoveJenny.zip)
>
> #### 使用方法 / Signature Usage
>
> 1. ##### 安装过期EV代码签名证书
>
> 2. ##### 编辑hook.ini，设置时间
>
>    **设置时间规则**：
>
>    - ###### 在证书有效期之内
>
>    - ###### 在证书被吊销之前
>
>      *(建议设置到**接近证书的生效日期**，因为很多证书都被吊销了)*
>
>    - ###### 建议2015-07-29前
>
>    ![设置时间](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230406174339.jpg)
>
> 3. ##### 打开工具DSignTool.exe
>
>    **会提示时间戳日期**：
>
>    ![20230406174916](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230406174916.jpg)
>
> 4. ##### 添加规则
>
>    ![20230406175056](Pictures/20230406175056.jpg)
>
> 5. ##### 签名文件
>
>    ![20230406175256](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230406175256.jpg)

### 其他工具：微软SignTool / Other tools: Microsoft SignTool CMD Usage
> #### 签名方法 / Signature Method
>
> ```shell
> signtool timestamp /t "http://<服务器地址>/{SHA1|SHA256}/YYYY-MM-DDTHH:mm:ss" <待签名程序>
> ```
>
> #### 签名示例 / Signature Example
>
> ```shell
> signtool timestamp /t "http://time.pika.net.cn/fake/RSA/SHA1/2011-01-01T00:00:00" test.exe
> signtool timestamp /tp 1 /tr "http://time.pika.net.cn/fake/RSA/SHA256/2011-01-01T00:00:00" test.exe
> ```

## 搭建服务 / TS Server

### VS编译构建时间戳服务器

- #### 下载项目

- #### 修改代码

- #### 编译构建

### Windows部署服务（推荐）



### Ubuntu部署服务（不推荐）

#### 安装Wine

```shell
sudo dpkg --add-architecture i386
sudo apt-get install wine mono-complete winetricks wine32 winbind
```

#### 安装.Net

- ##### 自动安装

  ```shell
  sudo winetricks dotnet45
  ```

- #### 手动安装

  1. 下载文件 [wine-mono-7.4.0-x86.msi](Download/wine-mono-7.4.0-x86.msi) 

  2. ```shell
     wine uninstaller
     wine64 uninstaller
     ```

     安装上一步下载的MSI文件(wine-mono-7.4.0-x86.msi)

     ![安装MSI文件](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230406171727.jpg)

     

- #### 运行服务

## 签名工具 / Sign Tool

### VS编译HookSigntool

- #### 下载项目

- #### 修改代码

- #### 编译构建

## 参考资料 / Reference

> [1] 时间戳签名库以及本地Demo服务器，可以倒填时间制造有效签名，JemmyloveJenny，吾爱破解，https://www.52pojie.cn/thread-908684-1-1.html
>
> [2] 亚洲诚信数字签名工具修改版 自定义时间戳 驱动签名，JemmyloveJenny，吾爱破解，https://www.52pojie.cn/thread-1027420-1-1.html
