---
title: Egress路由器配置方案
slug: "egress-router-setup"
---



## 场景需求

使用Egress router当作Proxy服务器，控制集群中只能通过此Proxy访问企业内部受到安全
审计的数据库

参考技术资料：
https://docs.openshift.com/container-platform/3.5/admin_guide/managing_networking.html


## 问题

https://github.com/openshift/origin/issues/17421


## 解决办法

1、需要手工创建macvlan0设备,设备名称必须为：macvlan0

```
ip link add macvlan0 link eth0 type macvlan mode bridge
```

2、发布egress router镜像Pod文件
但是官方参考资料描述不准确。
必须在POD中加上
```
hostNetwork: true
```

3、如果基础环境是Openstack网络，请注意加上白名单

```
neutron port-update $neutron_port_uuid \
  --allowed_address_pairs list=true \
  type=dict mac_address=<mac_address>,ip_address=<ip_address>
```

4、原理分析，egress这个镜像里面并没有自动创建macvlan设备，

 https://github.com/openshift/origin/blob/master/images/egress/router/egress-router.sh

 ```
 function setup_network() {
    # The pod may die and get restarted; only try to add the
    # address/route/rules if they are not already there.
    if ! ip route get "${EGRESS_GATEWAY}" | grep -q macvlan0; then
    ip addr add "${EGRESS_SOURCE}"/32 dev macvlan0
    ip link set up dev macvlan0

    ip route add "${EGRESS_GATEWAY}"/32 dev macvlan0
    ip route del default
    ip route add default via "${EGRESS_GATEWAY}" dev macvlan0
    fi

    # Update neighbor ARP caches in case another node previously had the IP. (This is
    # the same code ifup uses.)
    arping -q -A -c 1 -I macvlan0 "${EGRESS_SOURCE}"
    ( sleep 2;
      arping -q -U -c 1 -I macvlan0 "${EGRESS_SOURCE}" || true ) > /dev/null 2>&1 < /dev/null &
}
 ```
