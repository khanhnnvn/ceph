# Hướng dẫn cài đặt CEPH Luminous trên mô hình 03 node

## 1. Môi trường cài đặt

- CentOS 7.4 64bit
- CEPH Luminous 
- Phương thức sử dụng để triển khai ceph `ceph-deploy`
- Vai trò các node như sau:
  - CEPH Admin nodes: ceph1
	- CEPH MON nodes: ceph1, ceph2, ceph3
	- CEPH OSD nodes: ceph1, ceph2, ceph3
	
- Lưu ý: 	
  - Tùy vào kiến trúc mà ta có thể khai báo các nodes MON hoặc OSD là các node tách biệt nhau hoặc node MON chỉ cần 01 node duy nhất. 
  - Số node MON thường là 03 hoặc 05 hoặc số lẻ để đảm bảo cụm cluser theo nguyên tắc mà CEPH khuyến cáo. Chi tiết nguyên tắc này đọc thêm các tài liệu từ docs.ceph.com nhé.
  - Trong phạm vi bài viết này chúng ta sẽ setup các node có cả vai trò MON và OSD để biết các tham số cấu hình ở phần dưới. ` NÊN TUÂN THỦ ĐÚNG CÁC BƯỚC VÀ CẤU HÌNH KHUYẾN CÁO ĐỂ ĐẢM BẢO CÀI ĐẶT THÀNH CÔNG - SAU ĐÓ HÃY SÁNG TẠO.`  


## 2. Mô Hình

![topo_ceph3node.png](./images/topo_ceph3node.png)


## 3. IP Planning

![ip_planning_ceph03node.png](./images/ip_planning_ceph03node.png)

## 4. Các bước cài đặt

### 4.1. Thiết lập IP, hostname

#### 4.1.1. Thiết lập IP, hostname cho ceph1

-  Đăng nhập với tài khoản root

	```
	su -
	```

- Khai báo repos nếu có

	```sh
	echo "proxy=http://192.168.70.111:3142;" >> /etc/yum.conf
	```

- Update OS

	```sh
	yum update -y
	```

- Đặt hostname

	```sh
	hostnamectl set-hostname ceph1
	```

- Đặt IP cho node `ceph1`

	```sh
	echo "Setup IP  ens160"
	nmcli c modify ens160 ipv4.addresses 192.168.70.131/24
	nmcli c modify ens160 ipv4.gateway 192.168.70.1
	nmcli c modify ens160 ipv4.dns 8.8.8.8
	nmcli c modify ens160 ipv4.method manual
	nmcli con mod ens160 connection.autoconnect yes

	echo "Setup IP  ens192"
	nmcli c modify ens192 ipv4.addresses 192.168.82.131/24
	nmcli c modify ens192 ipv4.method manual
	nmcli con mod ens192 connection.autoconnect yes

	echo "Setup IP  ens224"
	nmcli c modify ens224 ipv4.addresses 192.168.83.131/24
	nmcli c modify ens224 ipv4.method manual
	nmcli con mod ens224 connection.autoconnect yes
	```

-  Cấu hình các thành phần cơ bản

	```sh
	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	```

- Khai báo file  /etc/hosts

	```sh
	echo "192.168.82.131 ceph1" >> /etc/hosts
	echo "192.168.82.132 ceph2" >> /etc/hosts
	echo "192.168.82.133 ceph3" >> /etc/hosts
	echo "192.168.82.139 cephclient1" >> /etc/hosts	


	echo "192.168.70.131 ceph1" >> /etc/hosts
	echo "192.168.70.132 ceph2" >> /etc/hosts
	echo "192.168.70.133 ceph3" >> /etc/hosts
	echo "192.168.70.139 cephclient1" >> /etc/hosts
	```

- Khởi động lại

	```
	init 6
	```

#### 4.1.2  Thiết lập IP, hostname cho `ceph2`

- Đăng nhập với tài khoản root

	```
	su -
	```

- Khai báo repos nếu có

	```sh
	echo "proxy=http://192.168.70.111:3142;" >> /etc/yum.conf
	```

- Update OS

	```sh 
	yum update -y
	````

- Đặt hostname

	```sh
	hostnamectl set-hostname ceph2
	```

- Đặt IP cho node `ceph2`

	```sh
	echo "Setup IP  ens160"
	nmcli c modify ens160 ipv4.addresses 192.168.70.132/24
	nmcli c modify ens160 ipv4.gateway 192.168.70.1
	nmcli c modify ens160 ipv4.dns 8.8.8.8
	nmcli c modify ens160 ipv4.method manual
	nmcli con mod ens160 connection.autoconnect yes

	echo "Setup IP  ens192"
	nmcli c modify ens192 ipv4.addresses 192.168.82.132/24
	nmcli c modify ens192 ipv4.method manual
	nmcli con mod ens192 connection.autoconnect yes

	echo "Setup IP  ens224"
	nmcli c modify ens224 ipv4.addresses 192.168.83.132/24
	nmcli c modify ens224 ipv4.method manual
	nmcli con mod ens224 connection.autoconnect yes
	```

- Cấu hình các thành phần cơ bản

	```sh
	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	```

- Khai báo file `/etc/hosts`

	```sh
	echo "192.168.82.131 ceph1" >> /etc/hosts
	echo "192.168.82.132 ceph2" >> /etc/hosts
	echo "192.168.82.133 ceph3" >> /etc/hosts
	echo "192.168.82.139 cephclient1" >> /etc/hosts	


	echo "192.168.70.131 ceph1" >> /etc/hosts
	echo "192.168.70.132 ceph2" >> /etc/hosts
	echo "192.168.70.133 ceph3" >> /etc/hosts
	echo "192.168.70.139 cephclient1" >> /etc/hosts
	```

- Khởi động lại

	```sh
	init 6
	```

#### 4.1.3  Thiết lập IP, hostname cho `ceph3`

-  Đăng nhập với tài khoản `root`

	```sh
	su -
	```

-  Khai báo repos nếu có

	```sh
	echo "proxy=http://192.168.70.111:3142;" >> /etc/yum.conf
	```

- Update OS

	```sh
	yum update -y
	```

- Đặt hostname

	```sh
	hostnamectl set-hostname ceph3
	```

- Đặt IP cho node `ceph3`

	```sh
	echo "Setup IP  ens160"
	nmcli c modify ens160 ipv4.addresses 192.168.70.133/24
	nmcli c modify ens160 ipv4.gateway 192.168.70.1
	nmcli c modify ens160 ipv4.dns 8.8.8.8
	nmcli c modify ens160 ipv4.method manual
	nmcli con mod ens160 connection.autoconnect yes

	echo "Setup IP  ens192"
	nmcli c modify ens192 ipv4.addresses 192.168.82.133/24
	nmcli c modify ens192 ipv4.method manual
	nmcli con mod ens192 connection.autoconnect yes

	echo "Setup IP  ens224"
	nmcli c modify ens224 ipv4.addresses 192.168.83.133/24
	nmcli c modify ens224 ipv4.method manual
	nmcli con mod ens224 connection.autoconnect yes
	```

- Cấu hình các thành phần cơ bản

	```sh
	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	```

-  Khai báo file `/etc/hosts`

	```sh
	echo "192.168.82.131 ceph1" >> /etc/hosts
	echo "192.168.82.132 ceph2" >> /etc/hosts
	echo "192.168.82.133 ceph3" >> /etc/hosts
	echo "192.168.82.139 cephclient1" >> /etc/hosts	


	echo "192.168.70.131 ceph1" >> /etc/hosts
	echo "192.168.70.132 ceph2" >> /etc/hosts
	echo "192.168.70.133 ceph3" >> /etc/hosts
	echo "192.168.70.139 cephclient1" >> /etc/hosts
	```

-  Khởi động lại

	```sh
	init 6
	```

### 4.2. Cài gói bổ trợ và tạo tài khoản để cài đặt CEPH

#### Lưu ý: Cài đặt gói cơ bản trên cả 03 node CEPH1, CEPH2 và CEPH3

- Thực hiện update OS và cài các gói bổ trợ

	```sh
	yum update -y

	yum install epel-release -y

	yum install wget bybo curl git -y

	yum install python-setuptools -y

	yum install python-virtualenv -y

	yum update -y
	```

- Cấu hình NTP

	```sh
	yum install -y ntp ntpdate ntp-doc

	ntpdate 0.us.pool.ntp.org

	hwclock --systohc

	systemctl enable ntpd.service
	systemctl start ntpd.service
	```

- `Lưu ý:` trường hợp máy chủ tại Nhân Hòa thì cần khai báo IP về NTP server, liên hệ đội RD để được hướng dẫn.

- Tạo user `cephuser` trên node `ceph1, ceph2, ceph3`

	```sh
	useradd -d /home/cephuser -m cephuser
	```

- Đặt password cho user `cephuser`

	```sh
	passwd cephuser
	```

- Cấp quyền sudo cho tài khoản `cephuser`

	```sh
	echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
	chmod 0440 /etc/sudoers.d/cephuser
	# sed -i s'/Defaults requiretty/#Defaults requiretty'/g /etc/sudoers
	```

### 4.3 Tạo repos để cài đặt CEPH bằng cách tạo file

- `Lưu ý:` Thực hiện trên tất cả 03 node `ceph1, ceph2 và ceph3`

- Tạo repos
```sh
cat << EOF > /etc/yum.repos.d/ceph.repo
[Ceph]
name=Ceph packages for \$basearch
baseurl=http://download.ceph.com/rpm-luminous/el7/\$basearch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1


[Ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-luminous/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1

[ceph-source]
name=Ceph source packages
baseurl=http://download.ceph.com/rpm-luminous/el7/SRPMS
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
priority=1
EOF
```

-  Thực hiện update sau khi khai báo repos 

	```sh
	yum update -y
	```

### 4.3 Cài đặt CEPH

#### 4.3.1 Cài đặt ceph-deploy trên `ceph1`

- `Lưu ý:` Bước này thực hiện trên node `ceph1`

- Cài đặt ceph-deploy

	```sh
	yum install -y ceph-deploy
	```

- Kiểm tra lại phiên bản của ceph-deploy, chính xác là phiên bản 2.0.1

	```sh
	ceph-deploy --version
	```

	- Kết quả: 

		```sh
		[cephuser@ceph1 ~]$ ceph-deploy --version
		2.0.1
		```

- Chuyển sang tài khoản `cephuser`

	```sh
	su - cephuser
	```

- Tạo keypair trên node ceph1

	```sh
	ssh-keygen
	```

-  Copy keypair sang các node để ceph-deploy sử dụng. Nhập yes và mật khẩu của từng node khi được hỏi.

	```sh
	ssh-copy-id cephuser@ceph1

	ssh-copy-id cephuser@ceph2

	ssh-copy-id cephuser@ceph3
	```

#### 4.3.2 Thực hiện cài đặt CEPH

- `Lưu ý:` Bước này thực hiện trên node `ceph1`
-  Tạo thư mục để chứa file cài đặt ceph

	```sh
	cd ~
	mkdir my-cluster
	cd my-cluster
	```

- Thiết lập cluster cho CEPH. Cú pháp của lệnh sẽ là `ceph-deploy new ten_mon_nodes`. 
- Do trong cấu hình này ta dùng 03 node `ceph1, ceph2, ceph3` làm node MON nên ta sẽ thực hiện lệnh bên dưới. Nếu bản chỉ có 01 node thì chỉ cần hostname của node đó.

	```sh
	ceph-deploy new ceph1 ceph2 ceph3
	```

- Kết quả của lệnh trên sẽ sinh ra các file dưới, kiểm tra bằng lệnh `ls -alh`

	```sh
	[cephuser@ceph1 my-cluster]$ ls -alh
	total 172K
	drwxrwxr-x 2 cephuser cephuser   75 Oct 16 23:16 .
	drwx------ 4 cephuser cephuser  116 Oct 16 23:16 ..
	-rw-rw-r-- 1 cephuser cephuser  421 Oct 16 23:31 ceph.conf
	-rw-rw-r-- 1 cephuser cephuser 163K Oct 16 23:36 ceph-deploy-ceph.log
	-rw------- 1 cephuser cephuser   73 Oct 16 23:16 ceph.mon.keyring
	```
	
- File `ceph.conf` sinh ra ở trên chứa các cấu hình cho cụm ceph cluster. Ta thực hiện thêm các khai báo dưới cho file `ceph.conf` này trước khi cài đặt các gói cần thiết cho ceph trên các node.
	
	```sh
	echo "public network = 192.168.82.0/24" >> ceph.conf
	echo "cluster network = 192.168.83.0/24" >> ceph.conf
	echo "osd objectstore = bluestore"  >> ceph.conf
	echo "mon_allow_pool_delete = true"  >> ceph.conf
	echo "osd pool default size = 3"  >> ceph.conf
	echo "osd pool default min size = 1"  >> ceph.conf
	```

-  Cài đặt các gói của CEPH trên các node, trong hướng dẫn này chỉ rõ bản `ceph luminous`. Lệnh dưới sẽ cài các gói lên tất cả các node ceph. Lệnh được thực hiện trên node `ceph1`.

	```sh
	ceph-deploy install --release luminous ceph1 ceph2 ceph3
	```
	
- Kết quả của lệnh trên sẽ hiển thị như bên dưới, trong đó có phiên bản của ceph được cài trên các node.

	```sh
	[ceph3][DEBUG ]
	[ceph3][DEBUG ] Complete!
	[ceph3][INFO  ] Running command: sudo ceph --version
	[ceph3][DEBUG ] ceph version 12.2.8 (ae699615bac534ea496ee965ac6192cb7e0e07c0) luminous (stable)
	```

-  Cấu hình MON 

	```sh
	ceph-deploy mon create-initial
	```

- Kết quả của lệnh trên sẽ sinh ra các file dưới, kiểm tra bằng lệnh `ls -alh`

	```sh
	[cephuser@ceph1 my-cluster]$ ls -alh
	total 412K
	drwxrwxr-x 2 cephuser cephuser  244 Oct 16 23:47 .
	drwx------ 4 cephuser cephuser  116 Oct 16 23:16 ..
	-rw------- 1 cephuser cephuser   71 Oct 16 23:47 ceph.bootstrap-mds.keyring
	-rw------- 1 cephuser cephuser   71 Oct 16 23:47 ceph.bootstrap-mgr.keyring
	-rw------- 1 cephuser cephuser   71 Oct 16 23:47 ceph.bootstrap-osd.keyring
	-rw------- 1 cephuser cephuser   71 Oct 16 23:47 ceph.bootstrap-rgw.keyring
	-rw------- 1 cephuser cephuser   63 Oct 16 23:47 ceph.client.admin.keyring
	-rw-rw-r-- 1 cephuser cephuser  421 Oct 16 23:31 ceph.conf
	-rw-rw-r-- 1 cephuser cephuser 191K Oct 16 23:47 ceph-deploy-ceph.log
	-rw------- 1 cephuser cephuser   73 Oct 16 23:16 ceph.mon.keyring
	````
		
- Thực hiện copy file `ceph.client.admin.keyring` sang các node trong cụm ceph cluster. File này sẽ được copy vào thư mục `/etc/ceph/`

	```sh
	ceph-deploy admin ceph1 ceph2 ceph3
	```

- Đứng trên node `ceph1` phân quyền cho file 	`/etc/ceph/ceph.client.admin.keyring`

	```sh
	sudo chmod +r /etc/ceph/ceph.client.admin.keyring
	```

- Tiếp tục ssh vào các node ceph2 và ceph3 còn lại để thực hiện lệnh phân quyền thực thi cho file `/etc/ceph/ceph.client.admin.keyring`

	```sh
	sudo chmod +r /etc/ceph/ceph.client.admin.keyring
	```

Việc trên có ý nghĩa là để có thể thực hiện lệnh quản trị của CEPH trên các 03 node trong cụm Cluster.

#### 4.3.3 Add các OSD cho cụm CEPH

- Add các OSD cho cụm ceph cluser
 

	```sh
	ceph-deploy osd create --data /dev/sdb ceph1

	ceph-deploy osd create --data /dev/sdc ceph1

	ceph-deploy osd create --data /dev/sdd ceph1


	ceph-deploy osd create --data /dev/sdb ceph2

	ceph-deploy osd create --data /dev/sdc ceph2

	ceph-deploy osd create --data /dev/sdd ceph2


	ceph-deploy osd create --data /dev/sdb ceph3

	ceph-deploy osd create --data /dev/sdc ceph3

	ceph-deploy osd create --data /dev/sdd ceph3
	```


#### 4.3.4 Cấu hình manager và dashboad cho ceph cluster


- Thực hiện trên node ceph1 để khai báo các node có vai trò manager, phục vụ việc quản trị ceph sau này.

	```
	ceph-deploy mgr create ceph1 ceph2 ceph3
	```

- Kích hoạt dashboad
	
	```sh
	ceph mgr module enable dashboard
	```

- Kiểm tra trạng thái của ceph dashboad và port để truy cập.

	```sh
	ceph mgr dump
	```

- Kết quả: http://prntscr.com/l58wm7

- Truy cập vào địa chỉ IP với port mặc định là 7000 như ảnh: `http://ip_address_ceph1:7000`. 

- Ta sẽ có giao diện như link: 
  - http://prntscr.com/l5k7xj
	- http://prntscr.com/l6ryli
	- http://prntscr.com/l6ryzp

#### 4.3.5 Kiểm tra lại hoạt động của CEPH

- Thực hiện lệnh dưới để kiểm tra trạng thái hoạt động của ceph

	```sh
	ceph -s 
	```

- Kết quả: 

	```sh
	[cephuser@ceph1 my-cluster]$ ceph -s
		cluster:
			id:     0789974d-1ebb-43bd-8084-c51dd08d7888
			health: HEALTH_OK

		services:
			mon: 3 daemons, quorum ceph1,ceph2,ceph3
			mgr: ceph1(active), standbys: ceph2, ceph3
			osd: 9 osds: 9 up, 9 in

		data:
			pools:   0 pools, 0 pgs
			objects: 0 objects, 0B
			usage:   9.04GiB used, 1.75TiB / 1.76TiB avail
			pgs:
	```

### 5. Cài đặt RBD cho client sử dụng

#### 5.1. Cài đặt trên node client 

- Thiết lập ip trên node `cephclient1`

-  Đăng nhập với tài khoản root

	```
	su -
	```

- Khai báo repos nếu có

	```sh
	echo "proxy=http://192.168.70.111:3142;" >> /etc/yum.conf
	```

- Update OS

	```sh
	yum update -y
	```

- Đặt hostname

	```sh
	hostnamectl set-hostname cephclient1
	```

- Đặt IP cho node `cephclient1`

	```sh
	echo "Setup IP  eth0"
	nmcli c modify eth0 ipv4.addresses 192.168.70.139/24
	nmcli c modify eth0 ipv4.gateway 192.168.70.1
	nmcli c modify eth0 ipv4.dns 8.8.8.8
	nmcli c modify eth0 ipv4.method manual
	nmcli con mod eth0 connection.autoconnect yes

	echo "Setup IP  eth1"
	nmcli c modify eth1 ipv4.addresses 192.168.82.139/24
	nmcli c modify eth1 ipv4.method manual
	nmcli con mod eth1 connection.autoconnect yes
	```

-  Cấu hình các thành phần cơ bản

	```sh
	sudo systemctl disable firewalld
	sudo systemctl stop firewalld
	sudo systemctl disable NetworkManager
	sudo systemctl stop NetworkManager
	sudo systemctl enable network
	sudo systemctl start network

	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
	sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
	```

- Khai báo file  /etc/hosts

	```sh
	echo "192.168.82.131 ceph1" >> /etc/hosts
	echo "192.168.82.132 ceph2" >> /etc/hosts
	echo "192.168.82.133 ceph3" >> /etc/hosts
	echo "192.168.82.139 cephclient1" >> /etc/hosts	


	echo "192.168.70.131 ceph1" >> /etc/hosts
	echo "192.168.70.132 ceph2" >> /etc/hosts
	echo "192.168.70.133 ceph3" >> /etc/hosts
	echo "192.168.70.139 cephclient1" >> /etc/hosts
	```
	
- Khởi động lại node client 

	```sh
	init 6
	```


#### 5.2. Cài đặt ceph client cho node `cephclient1`

- Thực hiện trên node `ceph1`
- Di chuyển vào thư mục chứa các file cấu hình của ceph hoặc chuyển sang user `cephuser` để thực hiện các bước tiếp theo

	```sh
	cd /home/cephuser/my-cluster/
	```

- Đứng trên node `ceph1` thực hiện cài đặt 

	```sh
	ceph-deploy install cephclient1 
	```
	
- Thực hiện deploy ceph cho node `cephclient1`
	
	```sh
	ceph-deploy admin cephclient1 
	```

#### 5.2. Cài đặt ceph client cho node `cephclient1`

Thực hiện trên node `cephclient1`

- Phân quyền cho file `/etc/ceph/ceph.client.admin.keyring`

```sh
/etc/ceph/ceph.client.admin.keyring
```






























