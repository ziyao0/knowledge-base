# V2ray安装说明

执行脚本安装：

```sh
bash <(curl -sL https://raw.githubusercontent.com/hijkpw/scripts/master/centos_install_v2ray.sh)
```

查看v2ray配置/运行状态：

~~~sh
bash <(curl -sL https://raw.githubusercontent.com/hijkpw/scripts/master/centos_install_v2ray.sh) info
~~~

v2ray管理命令：

 ~~~sh
systemctl start v2ray
systemctl restart v2ray
systemctl stop v2ray
systemctl status v2ray
 ~~~

卸载

~~~sh
bash <(curl -sL https://raw.githubusercontent.com/hijkpw/scripts/master/centos_install_v2ray.sh) uninstall
~~~

