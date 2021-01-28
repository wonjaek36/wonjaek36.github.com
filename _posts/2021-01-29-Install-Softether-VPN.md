---
layout: post
title: "Install Softether VPN on Linux"
date: 2021-01-29 02:00 +0900
tags: [linux, vpn]
---

### 1. Introduction 
&nbsp;  
[SoftEther VPN](https://www.softether.org/)은 오픈소스이며, Window, Linux에서 VPN 서버를 설치하여 사용할 수 있다. 여기서는 Linux에 설치하는 방법을 설명한다.

### 2. Environment
  * Ubuntu 18.04

### 3. Download
  * [SoftEther VPN Download Center](https://www.softether-download.com/en.aspx?product=softether) 접속
  * SoftEther VPN Server -> Linux -> Architecture에 맞추고 파일 링크 복사
  * 저는 운영체제 Ubuntu, 64bit 선택

{% highlight shell %}
$ mkdir VPN_SRC # Create a directory for VPN installation file
$ cd VPN_SRC
$ wget https://www.softether-download.com/files/softether/v4.34-9745-rtm-2020.04.05-tree/Linux/SoftEther_VPN_Server/64bit_-_Intel_x64_or_AMD64/softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz # Download Install files
$ tar -xvf softether-vpnserver-v4.34-9745-rtm-2020.04.05-linux-x64-64bit.tar.gz
{% endhighlight %}
  
### 4. Install
{% highlight shell %}
$ sudo -i # root 권한 
$ mv ~/VPN_SRC/vpnserver /etc/vpnserver
$ cd /etc/vpnserver && make
# 1(yes) 을 계속해서 입력(License 동의)

# Service 등록
$ tee /etc/systemd/system/vpnserver.service << EOF
> [Unit]
> Description=SoftEther VPN Server
> After=network.target

> [Service]
> Type=forking
> ExecStart=/etc/vpnserver/vpnserver start
> ExecStop=/etc/vpnserver/vpnserver stop

> [Install]
> WantedBy=multi-user.target
> EOF

# Service 실행
$ systemctl daemon-reload
$ systemctl start vpnserver
$ systemctl status vpnserver
● vpnserver.service - SoftEther VPN Server
   Loaded: loaded (/etc/systemd/system/vpnserver.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2021-01-26 06:44:02 UTC; 2 days ago
  Process: 4571 ExecStart=/etc/vpnserver/vpnserver start (code=exited, status=0/SUCCESS)
  ...
{% endhighlight %}

### 5. Configuration
* Admin Password 설정
{% highlight shell %}
$ cd /etc/vpnserver
$ ./vpncmd

... 
Hostname of IP Address of Destination: <Enter>
Specify Virtual Hub Name: <Enter>

VPN Server> ServerPasswordSet
ServerPasswordSet command - Set VPN Server Administrator Password
Please enter the password. To cancel press the Ctrl+D key.

Password: <Password>
Confirm input: <Password>
{% endhighlight %}

* Hub Create & Nat 설정
  * 여기서 팀 이름은 Team1으로 가정
{% highlight shell %}
VPN Server> HubCreate Team1
VPN Server> hub Team1
VPN Server/Team1> SecureNatEnable # Nat 설정
**VPN Server/Team1> SecureNatHostGet # 설정하고 싶다면 SecureNatHostSet**
SecureNatHostGet command - Get Network Interface Setting of Virtual Host of SecureNAT Function
Item       |Value
-----------+-----------------
MAC Address|MAC 주소
IP Address |IP 주소
Subnet Mask|SUBNET MASK
The command completed successfully.

**VPN Server/Team1> DhcpGet # 설정하고 싶다면 DhcpSet**
DhcpGet command - Get Virtual DHCP Server Function Setting of SecureNAT Function
Item                           |Value
-------------------------------+--------------
Use Virtual DHCP Function      |Yes
Start Distribution Address Band|IP 시작 주소
End Distribution Address Band  |IP 끝 주소
Subnet Mask                    |SUBNET MASK 
Lease Limit (Seconds)          |7200
Default Gateway Address        |GATEWAY 주소
DNS Server Address 1           |DNS 주소
DNS Server Address 2           |None
Domain Name                    |
Save NAT and DHCP Operation Log|Yes
Static Routing Table to Push   |
The command completed successfully.
{% endhighlight %}

* User 생성
{% highlight shell %}
**VPN Server/APL> UserCreate user1**
UserCreate command - Create User
Assigned Group Name:

User Full Name: User1

User Description:

The command completed successfully.

**VPN Server/APL> UserPasswordSet user1**
UserPasswordSet command - Set Password Authentication for User Auth Type and Set Password
Please enter the password. To cancel press the Ctrl+D key.

Password: ********
Confirm input: ********


The command completed successfully.
{% endhighlight %}

* 포트 삭제
  * SoftEther은 기본적으로 443, 992, 5555 포트를 사용
  * Nginx나 웹서버가 있을 경우 443은 충돌이 발생
  * 여기서는 삭제한다

{% highlight shell %}
**VPN Server/APL>ListenerList**
ListenerList command - Get List of TCP Listeners
Port Number|Status
-----------+---------
TCP 443    |Listening
TCP 992    |Listening
TCP 5555   |Listening
The command completed successfully.

**VPN Server/APL>ListenerDelete**
VPN Server/APL>ListenerDelete
ListenerDelete command - Delete TCP Listener
Port number of TCP/IP Listener: 1194

The command completed successfully.
{% endhighlight %}