# 本地时间戳+签名工具使用教程

1. 创建一个CA和时间戳证书，参考[XCA自制CA证书并签发时间戳证书](https://code.52pika.cn/index.php/archives/330/)

2. 放置证书文件到当前的运行目录内，需要参考下面的文件说明：

   - **TSA.crt 证书Base64编码**

   - **TSA.key 密钥Base64编码**

3. 双击[StartAll.exe](StartAll.exe)

注：[SignTime.exe](SignTime.exe)是系统自带的Signtool签名时间戳自定义工具