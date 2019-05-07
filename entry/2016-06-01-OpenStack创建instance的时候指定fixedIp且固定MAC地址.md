---
layout: post
title: OpenStack中创建instance的时候指定fixed IP同时指定对应的MAC地址 
category: openstack 
tags: [openstack]
---

- 在OpenStack中，fixed ip一般是在创建instance的时候分配的，如果有需求在新建instance时分配给其特定的fixed ip该怎么做呢？以下是通过neutron + nova CLI API的实现方法：

    - 使用neutron port-create创建所需的fixed ip同时绑定MAC地址

        ```
        root@instance-OpIwBCzGR3Lh:~# neutron port-create --fixed-ip ip_address=172.17.51.111 --mac-address fa:16:3e:d1:fd:8c prod-net
        Created a new port:
        | Field                 | Value                                                                                |
        | admin_state_up        | True                                                                                 |
        | allowed_address_pairs |                                                                                      |
        | binding:host_id       |                                                                                      |
        | binding:profile       | {}                                                                                   |
        | binding:vif_details   | {}                                                                                   |
        | binding:vif_type      | unbound                                                                              |
        | binding:vnic_type     | normal                                                                               |
        | device_id             |                                                                                      |
        | device_owner          |                                                                                      |
        | fixed_ips             | {"subnet_id": "f5e4dc2a-15e0-4f5e-bb46-c8a83df304b4", "ip_address": "172.17.51.111"} |
        | id                    | 4f10b515-0b06-47da-8bed-237e3c14123a                                                 |
        | mac_address           | fa:16:3e:d1:fd:8c                                                                    |
        | name                  |                                                                                      |
        | network_id            | de5e4604-e8c6-4e14-8f10-9f274ba53a01                                                 |
        | security_groups       | de857503-3898-4259-9c8e-e318e5c1aa70                                                 |
        | status                | DOWN                                                                                 |
        | tenant_id             | af637eb440514f3abbc785423bd4c2ec                                                     |
        ```

        - 命令中prod-net是你环境中对应的网络名，比如常见的<ext-net>，其他的参数就是字面意思
    - IP和MAC对应好后，就可以使用这个"port"（neutron中的概念）来创建instance了，示例：

        ```
        root@instance-OpIwBCzGR3Lh:~# nova boot --flavor small --image Ubunt.trusty.14.04.x86_64 --nic port-id=4f10b515-0b06-47da-8bed-237e3c14123a --key-name hebb test
        | Property                             | Value                                                            |
        | OS-DCF:diskConfig                    | MANUAL                                                           |
        | OS-EXT-AZ:availability_zone          | nova                                                             |
        | OS-EXT-SRV-ATTR:host                 | -                                                                |
        | OS-EXT-SRV-ATTR:hypervisor_hostname  | -                                                                |
        | OS-EXT-SRV-ATTR:instance_name        | instance-00000280                                                |
        | OS-EXT-STS:power_state               | 0                                                                |
        | OS-EXT-STS:task_state                | scheduling                                                       |
        | OS-EXT-STS:vm_state                  | building                                                         |
        | OS-SRV-USG:launched_at               | -                                                                |
        | OS-SRV-USG:terminated_at             | -                                                                |
        | accessIPv4                           |                                                                  |
        | accessIPv6                           |                                                                  |
        | adminPass                            | Jc4kLEUMHZka                                                     |
        | config_drive                         |                                                                  |
        | created                              | 2016-06-01T01:38:05Z                                             |
        | flavor                               | small (01)                                                       |
        | hostId                               |                                                                  |
        | id                                   | 45c0b587-ff84-488e-beeb-1848f49a7ae7                             |
        | image                                | Ubunt.trusty.14.04.x86_64 (e1b87754-53a1-44f8-bfa0-cb4f0271466c) |
        | key_name                             | hebb                                                             |
        | metadata                             | {}                                                               |
        | name                                 | test                                                             |
        | os-extended-volumes:volumes_attached | []                                                               |
        | progress                             | 0                                                                |
        | security_groups                      | default                                                          |
        | status                               | BUILD                                                            |
        | tenant_id                            | af637eb440514f3abbc785423bd4c2ec                                 |
        | updated                              | 2016-06-01T01:38:05Z                                             |
        | user_id                              | 1fc6eecbc57c4492b9613164a38ef1ca                                 |
        ```
        - 命令中`--nic`就是用来指定`neutron port`的，port-id就是上一条命令结果中的id,其他字段一目了然，具体可以查询`nova help boot`来查询，至此，指定fixed ip和MAC地址的instance就创建好了,在horizon页面看到的结果![如图](/static/img/blog/openstackNovaBootWithSpecificFixedIP.jpg)

    - 参考链接：[OpenStack官方文档](http://docs.openstack.org/user-guide/cli_cheat_sheet.html)
