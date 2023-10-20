# 自建时间戳服务器实现伪签名驱动证书
## Implementing Pseudo Signature with Self-Sign Timestamp Servers
![自建时间戳服务器实现伪签名驱动证书](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/2586728848.jpg)

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

- ### 微软内核模式驱动代码签名要求：

  ![20230425155145](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230425155145.jpg)

  | 适用于：       | Windows Vista、Windows 7;带安全启动的 Windows 8+ | Windows 8、Windows 8.1、Windows 10 版本 1507、1511 以及安全启动 | Windows 10 版本 1607、1703、1709 以及安全启动                | Windows 10 版本 1803 及安全启动                              |
  | :------------- | :----------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
  | **架构：**     | 仅 64 位，32 位不需要签名                        | 64 位、32 位                                                 | 64 位、32 位                                                 | 64 位、32 位                                                 |
  | **需要签名：** | 嵌入文件或目录文件                               | 嵌入文件或目录文件                                           | 嵌入文件或目录文件                                           | 嵌入文件或目录文件                                           |
  | **签名算法：** | SHA2                                             | SHA2                                                         | SHA2                                                         | SHA2                                                         |
  | **证书：**     | 代码完整性信任的标准根                           | 代码完整性信任的标准根                                       | Microsoft 根证书颁发机构 2010、Microsoft 根证书颁发机构、Microsoft 根证书颁发机构 | Microsoft 根证书颁发机构 2010、Microsoft 根证书颁发机构、Microsoft 根证书颁发机构 |

  - Windows 10 1607 及更新版本
    - 如果EV代码签名证书是2015/07/19及**以前**由微软交叉签名的CA颁发的证书：
      - （必须）签署EV代码签名证书：SHA2（SHA256，SHA1实测仍然可以）
    - 如果EV代码签名证书是2015/07/19及**以后**（微软交叉签名）CA颁发的证书：
      - （必须）签署微软WHQL签名：SHA1/SHA2（SHA256，SHA1目前可以）
      - （可选）签署EV代码签名证书：SHA2（SHA256，微软官方不允许SHA1）
  - Windows 10 1607 之前的版本（到Vista/Win7/Win8/8.1）
    - （必须）签署EV代码签名证书：SHA1/SHA2（SHA256，SHA1实测仍然可以）
    - **对于某些旧版Win7及之前的版本，可能不支持SHA256**

  - Windows XP/2000，更早的系统不要求内核驱动强制签名
    - （可选）普通代码签名证书：SHA1（不支持SHA256）

- ### 微软禁用内核驱动强制签名方法：

  - Windows 10 1607 之后UEFI引导模式，并且开启Secure Boot选项：

    - 无法开启测试模式，*不能通过修改BCD解决*，**可以使用EFIGuard**

    - 每次可以开机进入<u>高级模式</u>-选择<u>禁用内核驱动强制签名</u>启动

  - Windows 10 1607 之前，Win8/8.1/7/Vista，或者关闭Secure Boot：

    - **开启测试模式**： 

      ```shell
      bcdedit /enum all
      bcdedit /set {default} testsigning on
      bcdedit /set nointegritychecks on
      bcdedit /set testsigning on
      bcdedit /debug ON
      bcdedit /bootdebug ON
      ```

    - 也可以每次可以开机进入<u>高级模式</u>-选择<u>禁用内核驱动强制签名</u>启动

- ### 签名验证原理

  ![签名验证原理](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230425155746.jpg)

  - ##### 对二进制文件校验摘要（SHA1/SHA256）

  - ##### 检查摘要是否与记录的符合

  - ##### 使用公钥校验摘要的签名正确

  - ##### 校验签名的证书是否有效，是否为EV证书

  - ##### 逐级向上校验证书信任链是否有效

  - ##### 检查CA是否被信任

  - ##### 检查CA证书是否被微软驱动CA交叉签名

- ### 伪造签名原理

  ![伪造签名原理](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230425160222.jpg)

  - #### 自建伪造时间戳服务器

    - ##### 自建CA证书（CA=TRUE，密钥用法=Certificate Signing, Off-line CRL Signing, CRL Signing，增强型密钥用法=2.5.29.32.0）

    - ##### 自签时间戳签名证书（密钥用途=Digital Signature，增强型密钥用法=时间戳 ，OCSP-URL，CRL-URL）

    - ##### 设置CRL地址（推荐Nginx，把CRL文件放入对应地址），或者设置OCSP服务器（OpenSSL OCSP）

    - ##### 搭建并启动时间戳响应服务器（*RFC*3161以及Authenticode格式 ，需要同时支持SHA1+SHA256）

  - #### 修改签名工具 / 使用SignTool

    - ##### [亚洲诚信签名工具 / TrustAsia SignTool - PikaFakeTimers](https://github.com/PIKACHUIM/FakeSign/raw/main/SignTool/Released/HookSigntool-PikaFakeTimers.zip)

    - ##### [亚洲诚信签名工具 / TrustAsia SignTool - JemmyLoveJenny](https://github.com/PIKACHUIM/FakeSign/raw/main/SignTool/Released/HookSigntool-JemmyLoveJenny.zip)

  - #### 使用泄漏EV证书有效期之内伪造签名

  - #### 在运行驱动的设备信任时间戳CA证书

    - #### [自动安装工具（推荐）](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/pika-fake-root-cert.exe)

    - #### [注册安装工具（手动）](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/JemmyLoveJenny-cert.reg)

## 快速使用 / QuickUse

### 0.本地整合文件

整合包包括本地时间戳服务器和本地代码签名工具

如果选择本地整合包, 需要自己生成CA和TSA证书

此方法不用再下载签名工具和使用在线时间戳服务

您只需要泄露的证书和自建CA信任体系并安装CA

**只适用于Windows**

- 虚假时间签名工具整合包：[Signtool-Stamp-Fake.zip](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/Signtool-Stamp-Fake.zip)

- 真实时间签名工具整合包：[Signtool-Stamp-Real.zip](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/Signtool-Stamp-Real.zip)

### 1.证书机构信任

用于时间戳认证 / CA Certificate:  Used for Timestamp Auth

二选一，***需要和下面签名工具的时间戳证书一致***

- Pikachu Fake CA （推荐）：[自动安装工具（推荐）](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/pika-fake-root-cert.exe)

- JemmyLoveJenny（备用）：[注册安装工具（手动）](https://github.com/PIKACHUIM/FakeSign/raw/main/Download/JemmyLoveJenny-cert.reg)

### 2.签名代码证书

泄漏的过期签名代码证书 / Leaked Expired Signature Code Certificate

> ***需要2015-07-29及以前的EV代码签名证书***，**我不提供任何代码签名证书**
>
> ***EV code signing certificates from July 29, 2015 and earlier are required***,
>
> **I do NOT provide any code signing certificates**

### 3.代码签名工具

亚洲诚信数字签名工具包 / TrustAsia Digital Signature Toolkit Modify

#### 下载工具 / Download Tools

二选一，***需要和之前安装的CA证书一致***

 - ##### Pikachu Fake CA （推荐）：[亚洲诚信签名工具 / TrustAsia SignTool - PikaFakeTimers](https://github.com/PIKACHUIM/FakeSign/raw/main/SignTool/Released/HookSigntool-PikaFakeTimers.zip)

 - ##### JemmyLoveJenny（备用）：[亚洲诚信签名工具 / TrustAsia SignTool - JemmyLoveJenny](https://github.com/PIKACHUIM/FakeSign/raw/main/SignTool/Released/HookSigntool-JemmyLoveJenny.zip)

#### 使用方法 / Signature Usage

 1. ##### 安装过期EV代码签名证书

 2. ##### 编辑hook.ini，设置时间

    **设置时间规则**：`hook.ini`

    - ###### 在证书有效期之内

    - ###### 在证书被吊销之前

      *(建议设置到**接近证书的生效日期**，因为很多证书都被吊销了)*

    - ###### 建议2015-07-29前

      ```ini
      [Timestamp]
      Timestamp=2015-01-01T00:00:00
      ```

 3. ##### 打开工具DSignTool.exe

    **会提示时间戳日期**

 4. ##### 添加规则

    ![20230406175056](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20230406175056.jpg)

 5. ##### 签名文件

    ![20230406175256](https://github.com/PIKACHUIM/FakeSign/raw/main/Pictures/20231020165034.png)

### 其他签名工具

其他工具：微软SignTool / Other tools: Microsoft SignTool CMD Usage

#### 时间戳方法 / Signature Method

 ```shell
 signtool timestamp /t "http://<服务器地址>/{SHA1|SHA256}/YYYY-MM-DDTHH:mm:ss" <待签名程序>
 ```

#### 签名示例 / Signature Example

##### 单独加时间戳（默认添加第一个签名的时间戳）

 ```shell
 signtool timestamp /t "http://time.pika.net.cn/fake/RSA/SHA1/2011-01-01T00:00:00" test.exe
 ```

##### 单独加时间戳-指定签名序号

```shell
signtool timestamp /tp 1 /tr "http://time.pika.net.cn/fake/RSA/SHA256/2011-01-01T00:00:00" test.exe
```

##### SHA1签名+时间戳

```
signtool.exe sign /f Cert.pfx /p password /tr http://time.pika.net.cn/fake/RSA/2011-01-01T00:00:00 /as /v T.exe
```

##### SHA256签名+时间戳

```
signtool.exe sign /f Cert.pfx /p password /fd sha256 /tr http://time.pika.net.cn/fake/RSA/2011-01-01T00:00:00 /td sha256 /as /v T.exe
```

## 搭建服务 / TS Server

### VS编译构建时间戳服务器

- #### 下载项目

  ```shell
  git clone https://github.com/PIKACHUIM/FakeSign.git
  ```

- #### 修改代码

  编辑：`TSServer/Develop/TimeStamping/Program.cs`

  ```c#
  static readonly string supportFake = @"true";
  ```

  如果要使用伪造服务器，则此处填写`true`，如果要使用真实时间，应当填写`false`

- #### 编译构建

  输出：`TSServer/Develop/TimeStamping/bin/Debug`

### Windows部署服务（推荐）

1. 创建一个CA和时间戳证书，参考[XCA自制CA证书并签发时间戳证书](https://code.52pika.cn/index.php/archives/330/)
2. 放置证书文件到当前的运行目录内，需要参考下面的文件说明：
   - **TSA.crt 证书Base64编码**
   - **TSA.key 密钥Base64编码**
3. 双击：`TimeStamping.exe`即可运行


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

  创建一个CA和时间戳证书，参考[XCA自制CA证书并签发时间戳证书](https://code.52pika.cn/index.php/archives/330/)
  
  放置证书文件到当前的运行目录内，需要参考下面的文件说明：
  
  - **TSA.crt 证书Base64编码**
  - **TSA.key 密钥Base64编码**
  
  ```shell
  wine TimeStamping.exe
  ```

## 签名工具 / Sign Tool

### VS编译HookSigntool

- #### 下载项目

  ```shell
  git clone https://github.com/PIKACHUIM/FakeSign.git
  ```

- #### 修改代码

  编辑：`SignTool/Hooktool/main.cpp`

  ```c#
  if (!_wcsicmp(lpOriginalTS, L"{CustomTimestampMarker-SHA1}")) {
          wcscat(buf, L"http://time.pika.net.cn/fake/RSA/");
          wcscat(buf, lpTimestamp);
          return buf;
      }
      else if (!_wcsicmp(lpOriginalTS, L"{CustomTimestampMarker-SHA256}")) {
          wcscat(buf, L"http://time.pika.net.cn/fake/RSA/");
          wcscat(buf, lpTimestamp);
          return buf;
      }
  ```

  将里面的`http://time.pika.net.cn/fake/RSA/`修改为`http://你的地址/路径`

- #### 编译构建

  输出：`SignTool/Hooktool/bin/Debug`

## 参考资料 / Reference

> [1] 时间戳签名库以及本地Demo服务器，可以倒填时间制造有效签名，JemmyloveJenny，吾爱破解，https://www.52pojie.cn/thread-908684-1-1.html
>
> [2] 亚洲诚信数字签名工具修改版 自定义时间戳 驱动签名，JemmyloveJenny，吾爱破解，https://www.52pojie.cn/thread-1027420-1-1.html
>
> [3] 关于Windows驱动签名认证的大致总结，[ANY_LNK](https://space.bilibili.com/1337311595)，BiliBili，https://www.bilibili.com/read/cv17812616
>
> [4] 数字证书伪造与利用（仅方便用于驱动开发人员的调试，不得非法使用），
> MIAIONE，BiliBili，https://www.bilibili.com/read/cv9802857/