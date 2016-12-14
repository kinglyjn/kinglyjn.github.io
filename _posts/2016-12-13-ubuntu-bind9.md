---
layout: post
title:  "ubuntu14.04 bind9的安装配置"
desc: "ubuntu14.04 bind9的安装配置"
keywords: "bind9,ubuntu14.04,安装配置,kinglyjn,张庆力"
date: 2016-12-13
categories: [Linux]
tags: [Linux]
icon: fa-bookmark-o
---

### 服务端

#### bind9安装后目录文件列表

<img src="http://img.blog.csdn.net/20161213094036960?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2luZ2x5am4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast" style="width:70%;">

#### 安装bind9

```shell
#安装bind9 （dig @172.16.127.xxx version.bind chaos txt  
#配置完之后查看bind的版本）
sudo apt-get install bind9
```

#### 配置文件

```shell
sudo vi /etc/bind/named.conf.local //针对内网DNS域名解析
=======================================================
//
// Do any local configuration here
//
// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
zone "internal-sa" {
        type master;
        file "/etc/bind/zone-internal-sa/db.dns";//正向
};
zone “127.12.172.in-addr.arpa” {
        type master;
        notify no;
        file "/etc/bind/zone-internal-sa/db.reverse-dns"; //反向
};


//新建配置
/etc/bind/zone-internal-sa/db.dns
/etc/bind/zone-internal-sa/db.reverse-dns
ubuntu@dbserver:/etc/bind/zone-internal-sa$ vi db.dns 
------------------------------------------------------- 
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.internal-sa. root.internal-sa. (
                        2013102301      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.internal-sa.
ns1     IN      A       172.16.127.128

;configure
dbserver       IN      A      172.16.127.128
nimbusz        IN      A      172.16.127.129
supervisor01z  IN      A      172.16.127.130
supervisor02z  IN      A      172.16.127.131 


ubuntu@dbserver:/etc/bind/zone-internal-sa$ vi db.reverse-dns 
-------------------------------------------------------
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     ns1.internal-sa. root.internal-sa. (
                        2013102301      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.
;@       IN      NS      ns1.internal-sa.
;ns1     IN      A       172.16.127.128

;configure
128      IN      PTR     dbserver.
129      IN      PTR     nimbusz.
130      IN      PTR     supervisor01z.
131      IN      PTR     supervisor02z.


sudo vi /etc/bind/named.conf.options //针对外网DNS解析
=======================================================
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        forwarders {
                172.31.0.2;
    8.8.8.8;
    114.114.114.114;
        };

        //=====================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //=====================================================
        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};


//重启bind9服务
sudo service bind9 restart
```
<br>


### 客户端

```shell
# DHCP（Dynamic Host Configuration Protocol）配置（用bind9做DNS域名解析, 需要重启客户端）
sudo vi /etc/dhcp/dhclient.conf

=======================================================
# Configuration file for /sbin/dhclient, which is included in Debian's
#       dhcp3-client package.
#
# This is a sample configuration file for dhclient. See dhclient.conf's
#       man page for more information about the syntax of this file
#       and a more comprehensive list of the parameters understood by
#       dhclient.
#
# Normally, if the DHCP server provides reasonable information and does
#       not leave anything out (like the domain name, for example), then
#       few changes must be made to this file, if any.
#

option rfc3442-classless-static-routes code 121 = array of unsigned integer 8;

#send host-name "andare.fugue.com";
send host-name = gethostname();
#send dhcp-client-identifier 1:0:a0:24:ab:fb:9c;
#send dhcp-lease-time 3600;
#supersede domain-name "fugue.com home.vix.com";
#prepend domain-name-servers 127.0.0.1;
prepend domain-name-servers 172.16.127.xxx;
prepend domain-name "internal-sa ";                        //注意最后的空格！！！
request subnet-mask, broadcast-address, time-offset, routers,
        domain-name, domain-name-servers, domain-search, host-name,
        dhcp6.name-servers, dhcp6.domain-search,
        netbios-name-servers, netbios-scope, interface-mtu,
        rfc3442-classless-static-routes, ntp-servers,
        dhcp6.fqdn, dhcp6.sntp-servers;
#require subnet-mask, domain-name-servers;
#timeout 60;
#retry 60;
#reboot 10;


#查看bind9的状态：
netstat -ltnp
=======================================================
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      - 

#验证正向解析
ping 172.16.127.xxx

#验证反向解析
ubuntu@nimbusz:~$ host -t PTR  172.16.127.xxx                               
xxx.127.16.172.in-addr.arpa domain name pointer dbserver.

#查看bind的版本信息
ubuntu@nimbusz:~$ dig @172.16.127.128 version.bind chaos txt
xxx
;version.bind.           0       CH      TXT     "9.9.5-3ubuntu0.9-Ubuntu"
xxx
```
