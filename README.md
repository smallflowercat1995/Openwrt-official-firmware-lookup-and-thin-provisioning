# Openwrt 官方固件查找和精简配置

## 描述
这是一篇适合 arm64 设备的 Openwrt 官方固件查找和精简配置的文章，如果你想折腾，那就折腾  
稍微注意改改好能变成其他设备(比如x86)的教程，自己留着用还是很不错的

## 官网的固件查询下载
```
https://openwrt.org/toh/views/toh_fwdownload  
https://firmware-selector.openwrt.org  
网关：192.168.1.1  
无密码
```
## 更新源
```
opkg update
```
## (可选) 自动挂磁盘
```
opkg install block-mount blockd kmod-usb2 \
            kmod-usb3 kmod-usb-core kmod-usb-storage \
            kmod-fs-ext4 kmod-fs-msdos kmod-fs-nfs \
            kmod-fs-ntfs kmod-usb-storage-uas cfdisk \
            smartmontools blkid fdisk kmod-fs-btrfs btrfs-progs
```
## (可选) 安装 netdata 实时监控默认端口 :19999
```
opkg install netdata
```
## (可选) 安装 docker
```
opkg install docker docker-compose dockerd \
            luci-app-dockerman
```
## (可选) 安装 openclash
### 删除会影响安装的配置文件自己斟酌删还是不删，反正我是删，还是注释一下吧 ，免得一股脑全复制
```
# rm -rfv /etc/hotplug.d/ntp/25-dnsmasqsec
# rm -rfv /etc/init.d/dnsmasq
# rm -rfv /usr/lib/dnsmasq/dhcp-script.sh
# rm -rfv /usr/sbin/dnsmasq
# rm -rfv /usr/share/acl.d/dnsmasq_acl.json
# rm -rfv /usr/share/dnsmasq/dhcpbogushostname.conf
# rm -rfv /usr/share/dnsmasq/rfc6761.conf
# rm -rfv /etc/config/dhcp
# rm -rfv /etc/config/luci
```
### 安装 openclash 依赖包
```
opkg install libwolfssl coreutils-nohup bash \
            iptables dnsmasq-full curl \
            ca-certificates ipset ip-full \
            iptables-mod-tproxy iptables-mod-extra libcap \
            libcap-bin ruby ruby-yaml \
            kmod-tun kmod-inet-diag luci \
            luci-base coreutils jsonfilter \
            luci-compat
```
### 下载ipk https://github.com/vernesong/OpenClash/releases
```
# 下载方法失效？
# curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://github.com`curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k 'https://github.com/vernesong/OpenClash/releases' | grep ipk | sed 's;";  ;g' | awk '{print $3}' | head -n 1`" -O
DOWNLOAD=`curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k 'https://github.com/vernesong/OpenClash/releases' | sed 's;";\n;g;s;tag;download;g' | grep '/download/' | head -n 1`
curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://github.com${DOWNLOAD}/luci-app-openclash_`basename ${DOWNLOAD} | sed 's;v;;g'`_all.ipk" -o luci-app-openclash_`basename ${DOWNLOAD} | sed 's;v;;g'`_all.ipk -O
```
### 下载Dev clash核心 TUN clash核心 Meta clash核心 解压 https://github.com/vernesong/OpenClash/tree/core/dev
```
curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://raw.githubusercontent.com/vernesong/OpenClash/core/`curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k 'https://github.com/vernesong/OpenClash/tree/core/dev/dev/' | sed 's;";\n;g'  | grep arm64 | tail -n 1`" -o "clash.tar.gz"
curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://raw.githubusercontent.com/vernesong/OpenClash/core/`curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k 'https://github.com/vernesong/OpenClash/tree/core/dev/premium/' | sed 's;";\n;g'  | grep arm64 | tail -n 1`" -o "clash_tun.gz"
curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://raw.githubusercontent.com/vernesong/OpenClash/core/`curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k 'https://github.com/vernesong/OpenClash/tree/core/dev/meta/' | sed 's;";\n;g'  | grep arm64 | tail -n 1`" -o "clash_meta.tar.gz"
```
### 安装，安装完成后刷新LUCI页面，在菜单栏 -> 服务 -> OpenClash 进入插件页面
```
opkg install luci-app-openclash_*-beta_all.ipk
```
### 创建core目录
```
mkdir -p /etc/openclash/core/
```
### 解压
```
tar zxvf clash.tar.gz ; mv -v clash /etc/openclash/core/clash
gzip -dk clash_tun.gz ; mv -v clash_tun /etc/openclash/core/clash_tun
tar zxvf clash_meta.tar.gz ; mv -v clash /etc/openclash/core/clash_meta
chmod -R 755 /etc/openclash/core/
```
## （可选）安装usb转网口rj45
```
opkg install kmod-usb-net-asix kmod-usb-net-asix-ax88179
```
## (可选) 安装 ttyd 终端
```
opkg install ttyd luci-app-ttyd luci-i18n-ttyd-zh-cn
```
## (可选) 主题
### luci-theme-argon 背景图片上传路径  /overlay/upper/www/luci-static/argon/background/ /www/luci-static/argon/background/
```
# 下载方法失效？
# curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://github.com`curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k 'https://github.com/jerrykuku/luci-theme-argon/releases' | grep ipk | sed 's;";  ;g' | awk '{print $3}' | head -n 1`" -O
DOWNLOAD=`curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://github.com/jerrykuku/luci-theme-argon/releases" | sed 's;";\n;g' | grep "luci-theme-argon/tree" | sort -r | head -n 1  | sed 's;tree;releases/download;g'`
curl -SL --connect-timeout 30 -m 60 --speed-time 30 --speed-limit 1 --retry 2  -H "Connection: keep-alive" -k "https://github.com${DOWNLOAD}/luci-theme-argon_`basename ${DOWNLOAD} | sed 's;v;;g'`_all.ipk" -o luci-theme-argon_`basename ${DOWNLOAD} | sed 's;v;;g'`_all.ipk -O
```
## 安装主题
```
opkg install luci-theme-*.ipk
```
## (可选) 安装汉化包
```
opkg install luci-i18n-base-zh-cn
```
## reboot
```
reboot
```
## 手动挂载目录
```
# 查看磁盘信息命令 blkid 或者 fdisk -l
# 创建磁盘挂载目录比如我的设备路径为 /dev/sda1 则我可以为这个设备创建目录名字随意路径随意 mkdir -p /mnt/sda1
# 挂载磁盘 mount /dev/sda1 /mnt/sda1
# 进入软路由界面 系统 -> 挂载点  找到已经挂在的设备
# 检查无误后，在 挂载点 点击 新增 选择已启用， uuid 选择自己的设备路径， 挂载点 自定义填写 /mnt/sda1
# 点击 保存 会退出到 挂载点 页面，点击 保存并应用 即可
```
## 手动配置 docker 存储路径
```
# 进入软路由界面 docker -> 配置 将 Docker Root Dir 配置为自己想要存放的路径
# 点击 保存并应用 ，重启 docker 服务命令 /etc/init.d/dockerd restart
```
## 解决 openwrt 下 docker 无法联网的问题
```
# 首先 vim /etc/sysctl.conf 添加下列内容：
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.bridge.bridge-nf-call-arptables = 1
# 后台修改设置：
    Luci>网络>防火墙>转发：接受
    Luci>状态>防火墙>重启防火墙
# 最后ssh执行service dockerd restart
```
## 关闭防火墙，不建议，但是如果你受够了
```
iptables -F
iptables -X
/etc/init.d/firewall disable
```
## 如果要执行卸载命令按照以下卸载opencllash的例子即可
```
# 插件在卸载后会自动备份配置文件到 /tmp 目录下，除非路由器重启，在下次安装时将还原您的配置文件
# opkg remove luci-app-openclash
```
## 更新软件包列表
```
########################## 注意谨慎使用，最好别用 ################################

opkg update

## 查看更新数量
opkg list-upgradable | grep -v Multiple | awk '{print $1}' | wc -l

## 更新软件包
num=1
for i in `opkg list-upgradable | grep -v Multiple | awk '{print $1}'`; do
echo "==========================>>"$num--$i"<<=========================="
num=$((num+1))
opkg upgrade $i --force-depends
done
unset num

## sysupgrade 手动升级固件 会清除配置 
sysupgrade *-squashfs-sysupgrade.img.gz
```
## 问题
### 1、解决openwrt内核版本不一致,报错: cannot find dependency kernel (= 5.10.115-1-19cc8260705cc8ea5062882916b83dcc)
```
# 假设安装软件时提示需要5.10.115版本的内核，然而，系统的内核是5.17.12
uname -a
    -- Linux OpenWrt 5.17.12-flippy-73+ #50 SMP PREEMPT Mon May 30 19:23:09 CST 2022 aarch64 GNU/Linux
# 下载内核更新包，在官网的包库中找到内核更新包，获取链接
cat /etc/opkg/distfeeds.conf  | grep core | awk '{print $3}'

# 打开浏览器，搜索 kernel 进行搜索，然后找到了这个安装包 kernel_5.10.115-1-19cc8260705cc8ea5062882916b83dcc_aarch64_cortex-a53.ipk 下载
var=5.10.115-1-19cc8260705cc8ea5062882916b83dcc_aarch64
wget --verbose --show-progress=on --progress=bar --hsts-file=/tmp/wget-hsts -c "https://downloads.openwrt.org/snapshots/targets/armvirt/64/packages/kernel_${var}_aarch64_cortex-a53.ipk" -O kernel_${var}_aarch64_cortex-a53.ipk

# 安装
opkg install kernel_5.10.115-1-19cc8260705cc8ea5062882916b83dcc_aarch64_cortex-a53.ipk

# 重启
reboot
```
## 感谢
github vernesong 大佬

## 参考
https://github.com/vernesong/OpenClash/tree/master/core-lateset  
https://github.com/vernesong/OpenClash/releases  
https://www.right.com.cn/forum/thread-1209004-1-1.html  
https://www.right.com.cn/FORUM/thread-350773-1-1.html  
https://openwrt.org/zh-cn/doc/howto/storage  
