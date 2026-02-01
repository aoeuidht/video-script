# RouterOS Gateway Auto Switch

## Context

* [Youtube](https://youtu.be/ECGw6e3IbpU)
* [Bilibili](https://www.bilibili.com/video/BV1JpUfBpE4o/)

## Manual

* Notice: Run in the cli of __RouterOS__
* If the network speed is too slow, try
  - Disable all rules like `chain=input action=drop` in  `/ip/firewall/filter/print` for debugging, or
  - MASQUERADE traffic for LAN

### Edit default route distance

```
/ip/dhcp-client/set ether1 default-route-distance=25
```

### Route the traffic to OpenWRT

#### Create new route table

```
/routing/table/add name="rtab-op" fib
```

#### Add mangle rule

* Replace the ip `192.168.88.119` with your IP address or [IP address list](http://10.11.12.1/webfig/?radio1=on&radio2=on#IP:Firewall.Address_Lists)
```
/ip/firewall/mangle/add chain=prerouting action=mark-routing new-routing-mark=rtab-op passthrough=yes src-address=10.11.12.119 dst-address-type=!local log=no log-prefix=""
```

#### Create new routing

* Replace the IP `10.11.12.10` to the address of OpenWRT
```
# Route all traffic to OpenWRT
/ip/route/add dst-address=0.0.0.0/0 routing-table=rtab-op gateway=10.11.12.10 distance=20 suppress-hw-offload=no
# Route the checking traffic to Openwrt
/ip/route/add dst-address=1.1.1.1/32 routing-table=main gateway=10.11.12.10 distance=15 suppress-hw-offload=no
```

### Auto Recovery

#### Create Scripts

```
/system/script/add name="enable-op" owner="admin" dont-require-permissions=yes source="/ip/firewall/mangle set [find comment=\"mark-dev\"] disabled=no"
/system/script/add name="disable-op" owner="admin" dont-require-permissions=yes source="/ip/firewall/mangle set [find comment=\"mark-dev\"] disabled=yes"
```

#### Set Net-watch

```
/tool/netwatch/add host=1.1.1.1 type=https-get up-script=enable-op down-script=disable-op http-codes=100-599
```
