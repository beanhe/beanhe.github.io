---
layout: post
title: Implementing-Hight-Availability-Instances-with-Neutron-using-VRRP
category: openstack
tags: [openstack,VIP,keepalived,HA]
---
## Implementing High Availability Instances with Neutron using VRRP
#### 转自[arosen的post](http://blog.aaronorosen.com/implementing-high-availability-instances-with-neutron-using-vrrp/)
- In the Havana release we added a new extension called “Allowed-Address-Pairs” that allows one to add additional ips (or cidrs) with their mac-address to a port to allow traffic that matches those values to pass through. This was needed because by default neutron ports only allow traffic through that match the mac-address and fixed-ips fields on a port (which is done to enforce anti-spoofing).  Because of this, there was no way to support protocols such as VRRP which require mapping the same ip-address to multiple ports which neutron does not allow by design.

- In this post I’m going to demo how to use the allowed-address-pairs extension which is currently supported in the following plugins: OVS, ML2, VMware NSX, BigSwitch, and NEC. Then, demo it working with VRRP using keepalived.

- Just to give a little background how VRRP (Virtual Routing Redundancy Protocol) works before we dive in.  VRRP works by having hosts participate in a group which share a configured  ip-address. One node is elected master and is the only node that will respond on that giving address. The other nodes are slaves and just monitor the current master node periodically to ensure that it’s still running.  If the master node goes down one of the slave nodes take over as the master and start replying on the specified ip-address. The nice thing about this is it allows one to avoid a single point of failure in the system and doesn’t require a load-balancer if you’re not trying to provide scale out (though VRRP is often used in conjunction with load-balancers to provide additional HA and scale-out at the same time).

- To get started I’m assuming you already have an OpenStack deployment up and running that is Havana or newer with one of the supported plugins that implement allowed-address-pairs. If you’re unsure if you have this you can find out by querying neutron for the extensions it has loaded and looking for the allowed-address-pairs extension:

    ```
    $ neutron ext-list
    | alias                 | name                                          |
    | network-gateway       | Neutron-NVP Network Gateway                   |
    | security-group        | security-group                                |
    | dist-router           | Distributed Router                            |
    | router                | Neutron L3 Router                             |
    | mac-learning          | MAC Learning                                  |
    | port-security         | Port Security                                 |
    | ext-gw-mode           | Neutron L3 Configurable external gateway mode |
    | binding               | Port Binding                                  |
    | quotas                | Quota management support                      |
    | agent                 | agent                                         |
    | dhcp_agent_scheduler  | DHCP Agent Scheduler                          |
    | external-net          | Neutron external network                      |
    | multi-provider        | Multi Provider Network                        |
    | allowed-address-pairs | Allowed Address Pairs                         |
    | extraroute            | Neutron Extra Route                           |
    | provider              | Provider Network                              |
    | nvp-qos               | nvp-qos                                       |
    ```
- First, we’ll create a network that we’re going to put theses host on:

    ```
    $ neutron net-create vrrp-net
    ```

- Next, we’re going to attach a subnet to that network with a specified allocation-pool range. The reason we do this is we want to manually allocate addresses after 10.0.0.200 to use specifically for VRRP and we don’t want neutron to allocate those automatically.

    ```
    $ neutron subnet-create  --name vrrp-subnet --allocation-pool start=10.0.0.2,end=10.0.0.200 vrrp-net 10.0.0.0/24
    ```

- Then, we’re going to go ahead and create a router, uplink the vrrp-subnet to it, and attach the router to an upstream network called public:

    ```
    $ neutron router-create router1
    $ neutron router-interface-add router1 vrrp-subnet 
    $ neutron router-gateway-set router1 public
    ```

- Now, we’re going to create a security group called vrrp-sec-group and we’ll add rules to allow icmp, tcp port 80 and 22 ingress:

    ```
    $ neutron security-group-create vrrp-sec-group
    $ neutron security-group-rule-create  --protocol icmp vrrp-sec-group 
    $ neutron security-group-rule-create  --protocol tcp  --port-range-min 80 --port-range-max 80 vrrp-sec-group 
    $ neutron security-group-rule-create  --protocol tcp  --port-range-min 22 --port-range-max 22 vrrp-sec-group
    ```

- Next, we’re going to boot two instances using a ubuntu-12.04  image attached to the vrrp-net network (which is this uuid – 24e92ee1-8ae4-4c23-90af-accb3919f4d1) and vrrp-sec-group security group.

    ```
    nova boot --num-instances 2 --image ubuntu-12.04 --flavor 1 --nic net-id=24e92ee1-8ae4-4c23-90af-accb3919f4d1 vrrp-node --security_groups vrrp-sec-group
    ```

- The instances:

    ```
    $ nova list
    | ID                                   | Name                                            | Status | Task State | Power State | Networks                                               |
    | 15b70af7-2628-4906-a877-39753082f84f | vrrp-node-15b70af7-2628-4906-a877-39753082f84f | ACTIVE | -          | Running     | vrrp-net=10.0.0.3                                      |
    | e683e9d1-7eea-48dd-9d3a-a54cf9d9b7d6 | vrrp-node-e683e9d1-7eea-48dd-9d3a-a54cf9d9b7d6 | ACTIVE | -          | Running     | vrrp-net=10.0.0.4                                      |
    ```

- Create a port in the VRRP ip range that we left out of the ip-allocation range:

    ```
    $ neutron port-create --fixed-ip ip_address=10.0.0.201 --security-group vrrp-sec-group vrrp-net
    Created a new port:
    | Field                 | Value                                                                             |
    | admin_state_up        | True                                                                              |
    | allowed_address_pairs |                                                                                   |
    | device_id             |                                                                                   |
    | device_owner          |                                                                                   |
    | fixed_ips             | {"subnet_id": "94a0c371-d37c-4796-821e-57c2a8ec65ae", "ip_address": "10.0.0.201"} |
    | id                    | 6239f501-e902-4b02-8d5c-69062896a2dd                                              |
    | mac_address           | fa:16:3e:20:67:9f                                                                 |
    | name                  |                                                                                   |
    | network_id            | 24e92ee1-8ae4-4c23-90af-accb3919f4d1                                              |
    | port_security_enabled | True                                                                              |
    | security_groups       | 36c8131f-d504-4bcc-b708-f330c9f6b67a                                              |
    | status                | DOWN                                                                              |
    | tenant_id             | d4e4332d5f8c4a8eab9fcb1345406cb0                                                  |
    ```

- As you can see we were allocated a port with the ip-address 10.0.0.201 which we requested. We’ll also associate a floatingip to this port at this time so we can access it publicly:

    ```
    $ neutron floatingip-create --port-id=6239f501-e902-4b02-8d5c-69062896a2dd public
    Created a new floatingip:
    | Field               | Value                                |
    | fixed_ip_address    | 10.0.0.201                           |
    | floating_ip_address | 10.36.12.139                         |
    | floating_network_id | 3696c581-9474-4c57-aaa0-b6c70f2529b0 |
    | id                  | a26931de-bc94-4fd8-a8b9-c5d4031667e9 |
    | port_id             | 6239f501-e902-4b02-8d5c-69062896a2dd |
    | router_id           | 178fde65-e9e7-4d84-a218-b1cc7c7b09c7 |
    | tenant_id           | d4e4332d5f8c4a8eab9fcb1345406cb0     |
    ```

- We now need to update the ports attached to our VRRP instances to include this ip-address as an allowed-address-pair so they will be able to send traffic out using this address.  You’ll first need to find the ports attached to these instances:

    ```
    $ neutron port-list -- --network_id=24e92ee1-8ae4-4c23-90af-accb3919f4d1
    | id                                   | name | mac_address       | fixed_ips                                                                         |
    | 12bf9ea4-4845-4e2c-b511-3b8b1ad7291d |      | fa:16:3e:7a:7b:18 | {"subnet_id": "94a0c371-d37c-4796-821e-57c2a8ec65ae", "ip_address": "10.0.0.4"}   |
    | 14f57a85-35af-4edb-8bec-6f81beb9db88 |      | fa:16:3e:2f:7e:ee | {"subnet_id": "94a0c371-d37c-4796-821e-57c2a8ec65ae", "ip_address": "10.0.0.2"}   |
    | 6239f501-e902-4b02-8d5c-69062896a2dd |      | fa:16:3e:20:67:9f | {"subnet_id": "94a0c371-d37c-4796-821e-57c2a8ec65ae", "ip_address": "10.0.0.201"} |
    | 87094048-3832-472e-a100-7f9b45829da5 |      | fa:16:3e:b3:38:30 | {"subnet_id": "94a0c371-d37c-4796-821e-57c2a8ec65ae", "ip_address": "10.0.0.1"}   |
    | c080dbeb-491e-46e2-ab7e-192e7627d050 |      | fa:16:3e:88:2e:e2 | {"subnet_id": "94a0c371-d37c-4796-821e-57c2a8ec65ae", "ip_address": "10.0.0.3"}   |
    ```

- Add this address to the ports c080dbeb-491e-46e2-ab7e-192e7627d050, 12bf9ea4-4845-4e2c-b511-3b8b1ad7291d which are (10.0.0.3 and 10.0.0.4 our vrrp-node instances).

    ```
    $ neutron port-update  c080dbeb-491e-46e2-ab7e-192e7627d050 --allowed_address_pairs list=true type=dict ip_address=10.0.0.201
    $ neutron port-update  12bf9ea4-4845-4e2c-b511-3b8b1ad7291d --allowed_address_pairs list=true type=dict ip_address=10.0.0.201
    ```

- The allowed-address-pair 10.0.0.201 now shows up on the port:

    ```
    $ neutron port-show 12bf9ea4-4845-4e2c-b511-3b8b1ad7291d
    | Field                 | Value                                                                           |
    | admin_state_up        | True                                                                            |
    | allowed_address_pairs | {"ip_address": "10.0.0.201", "mac_address": "fa:16:3e:7a:7b:18"}                |
    | device_id             | e683e9d1-7eea-48dd-9d3a-a54cf9d9b7d6                                            |
    | device_owner          | compute:None                                                                    |
    | fixed_ips             | {"subnet_id": "94a0c371-d37c-4796-821e-57c2a8ec65ae", "ip_address": "10.0.0.4"} |
    | id                    | 12bf9ea4-4845-4e2c-b511-3b8b1ad7291d                                            |
    | mac_address           | fa:16:3e:7a:7b:18                                                               |
    | name                  |                                                                                 |
    | network_id            | 24e92ee1-8ae4-4c23-90af-accb3919f4d1                                            |
    | port_security_enabled | True                                                                            |
    | security_groups       | 36c8131f-d504-4bcc-b708-f330c9f6b67a                                            |
    | status                | ACTIVE                                                                          |
    | tenant_id             | d4e4332d5f8c4a8eab9fcb1345406cb0                                                |
    ```

- Login into our instances and configure VRRP first by installing keepalived:

    ```
    sudo apt-get install keepalived
    ```

- Now, we’ll need to configure keepalived. We’ll need to pick one of our nodes to be the master and the other as slave (though you can have multiple slave nodes if you want).
- Master configuration:

    ```
    $ cat  /etc/keepalived/keepalived.conf
    vrrp_instance vrrp_group_1 {
    state MASTER
    interface eth0
    virtual_router_id 1
    priority 100
    authentication {
    auth_type PASS
    auth_pass password
    }
    virtual_ipaddress {
    10.0.0.201/24 brd 10.0.0.255 dev eth0
    }
    }
    ```

- Slave Configuration:

    ```
    $ cat /etc/keepalived/keepalived.conf
    vrrp_instance vrrp_group_1 {
    state BACKUP
    interface eth0
    virtual_router_id 1
    priority 50
    authentication {
    auth_type PASS
    auth_pass password
    }
    virtual_ipaddress {
    10.0.0.201/24 brd 10.0.0.255 dev eth0
    }}
    ```
- restart keepalived to load the configuration change on both nodes:

    ```
     service keepalived restart
    ```
- Install a simple web-server to demo this working:

    ```
    sudo apt-get install apache2
    ```

- Run the following command to change the server response on each respective node:

    ```
    $ sudo echo "VRRP-node1" > /var/www/index.html
    $ sudo echo "VRRP-node2" > /var/www/index.html
    ```

- Issuing curl against the floatingip attached to the instance shows:

    ```
    $ curl 10.36.12.139
    VRRP-node1
    ```

- Now, we’ll induce a failure by setting the admin-state-up value on the port to False to show the fail over automatically working:

    ```
    $ neutron port-update 12bf9ea4-4845-4e2c-b511-3b8b1ad7291d --admin_state_up=False
    ```

- Curl shows our slave VRRP node responding:

    ```
    $ curl 10.36.12.139
    VRRP-node2
    ```

- And failing back over:

    ```
    $ neutron port-update 12bf9ea4-4845-4e2c-b511-3b8b1ad7291d --admin_state_up=True
    $ neutron port-update  c080dbeb-491e-46e2-ab7e-192e7627d050 --admin_state_up=False
    $ curl  10.36.12.139
    VRRP-node1
    ```
- In this example I created a special port to use with VRRP.  The only reason I did that is for flexibility so one could have a dedicated address on the VRRP instances and a special address that’s used to float between the instances. One could have also reused one of the addresses from one of the VRRP instances and mapped that as an allowed-address-pair instead.
- If you made it this far hopefully you found this useful :) . As always comments and questions welcome!
- VRRP config shamelessly stolen from: [http://archive09.linux.com/feature/114005](http://archive09.linux.com/feature/114005)

