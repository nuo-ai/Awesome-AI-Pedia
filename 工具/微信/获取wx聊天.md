# mac 解析内容 4.X版本的



下载个微信 的4.1.7的版本
https://github.com/zsbai/wechat-versions/releases?page=2



然后
git clone https://github.com/hicccc77/WeFlow

git reset --hard 93d46a3



```js
 pgrep -x WeChat || (open -a WeChat && sleep 4 && pgrep -x WeChat)     


```
/XXX/WeFlow/ 就是上面你安装WeFlow的路径
```js
cd /XXX/WeFlow/resources/key/macos/universal && sudo ./xkey_helper $(pgrep -x WeChat)
```
扫码登录 

就看到key了 ，

拿到key 后你直接让ai帮你去查，就能查到你的聊天记录了