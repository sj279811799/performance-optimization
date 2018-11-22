# 从 Cookie 到 Web Storage、IndexDB

## Cookie

Cookie以键值对形式存在。

### 性能优劣

Cookie最大只有4KB。Cookie是紧跟域名的，每次请求都会携带Cookie。过大的Cookie会在城性能浪费。

## Web Storage

由Local Storage和Session Storage组成。存储量大5~10M，位于浏览器端，不与服务端通信。

### Local Storage

Local Storage的特点是持久，一般用来存储不经常改变的base64图片和css、js。

### Session Storage

适合存储生命周期和它同步的会话级别的信息。

## IndexDB

运行在浏览器上的非关系型数据库。理论上没有存储上限。可以创建多个数据库，多张表。
