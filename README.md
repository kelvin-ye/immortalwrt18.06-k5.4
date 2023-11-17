  REPO_URL: https://github.com/immortalwrt/immortalwrt
  REPO_BRANCH: openwrt-18.06-k5.4


#更改lan接口后，会导致wifi无法启动
#  wifi不自启动，最有效方法（每次修改设置都要这样改一次）：
lan接口的“物理设置”，勾选“桥接接口”，勾选并在“自定义接口”填入：“ra0 rai0”，勾选“交换机 VLAN: "eth0.1" (lan)”

#  其它方法
# wifi不自启动，方法一：
临时的解决办法是不要在LAN界面修改任何信息。登录SSH，用vim /etc/config/network修改lan里需要的信息。

# wifi不自启动，方法二，可以启动，但是无法自动获取IP：
#ra0是2.4G，rai0是5G。
尝试在系统->启动项->本地启动脚本的exit 0前面添加以下脚本

#Wireless starter and Interface checker by Kenvix <i@kenvix.com>

#Start ra unconditionally
sleep 2s
ip link set rai0 up
ip link set ra0 up
echo "Wireless interface started"

#Lan Check
lanCheck=`uci get network.lan.ifname`
if [ $? -eq 0 ]; then
    echo $lanCheck | grep rai0 > /dev/null
    if [ $? -ne 0 ]; then
        uci set network.lan.ifname="$lanCheck rai0 ra0"
        uci commit
        echo "Updated wireless config of LAN Interface"
    fi
    echo "No need to update wireless config of LAN Interface"
else
    echo "wireless config of LAN Interface check failed. Interface may renamed." >&2
fi

# wifi不自启动，方法三，可以启动，但是无法自动获取IP：

在启动项那里加上
#启动2.4g 和 5g 信号
ip link set ra0 up
ip link set rai0 up

#桥接网卡
brctl addif br-lan ra0
brctl addif br-lan rai0

