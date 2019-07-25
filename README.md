# LEMP stack configuration files for CentOS

The **lempstack_centos** repository provides configuration files for setting up a secure LEMP web server on CentOS.

This is a port from the original [lempstack_debian](https://github.com/dommyet/lempstack_debian) repository.

## Repository Configurations

In this section we are going to install EPEL-release, [ELRepo Repository](http://elrepo.org/tiki/tiki-index.php), the [nginx Repository](http://nginx.org/en/linux_packages.html#RHEL-CentOS), and the [Remi's RPM Repository](https://rpms.remirepo.net/).

### EPEL-release

To install EPEL-release:

```
sudo yum install epel-release
```

### ELRepo

Import the public key, and install ELRepo for RHEL-7, SL-7 or CentOS-7:

```
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
sudo yum install https://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
```

### nginx Repository

Install the prerequisites:

```
sudo yum install yum-utils
```

To set up the yum repository, create the file named  `/etc/yum.repos.d/nginx.repo` with the following contents:

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
```

By default, the repository for stable nginx packages is used. If you would like to use mainline nginx packages, run the following command:

```
sudo yum-config-manager --enable nginx-mainline
```

To install nginx, run the following command:

```
sudo yum install nginx
```

### Remi's RPM Repository

We would install  `php73` in  `multiple versions simultaneously` mode in CentOS 7.

For more information, please refer to the [Configuration wizard](https://rpms.remirepo.net/wizard/).

Command to install the Remi repository configuration package:

```
sudo yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
```

Command to install the php73 collection:

```
sudo yum install php73
```

Command to install additional packages:

```
sudo yum install php73-php-fpm
```

Command to check the installed version and available extensions:

```
php73 --version
php73 --modules
```

## Initial Configurations

### Kernel

To install kernel-ml:

```
sudo yum --enablerepo=elrepo-kernel install kernel-ml
```

Change default kernel to kernel-ml:

```
sudo grub2-set-default 0
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

Remove old unused kernel automatically:

```
sudo yum install yum-utils
sudo package-cleanup --oldkernels --count=1
```

List all installed kernels:

```
sudo rpm -q kernel
```

### Congestion Control Algorithm Settings

To enable BBR:

```
echo 'net.core.default_qdisc=fq' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_congestion_control=bbr' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Confirm that BBR is enabled:

```
sysctl net.ipv4.tcp_available_congestion_control
```

The output should resemble:

```
net.ipv4.tcp_available_congestion_control = bbr cubic reno
```

Next, verify with:

```
sysctl -n net.ipv4.tcp_congestion_control
```

The output should be:

```
bbr
```

Finally, check that the kernel module was loaded:

```
lsmod | grep bbr
```

The output will be similar to:

```
tcp_bbr                16384  0
```

### Hostname

To show system hostname:

```
hostnamectl
```

To change system hostname:

```
sudo hostnamectl set-hostname <FQDN>
```

### SSH Authorized Keys Settings

Replace the default private key with your own key located in:

```
~/.ssh/authorized_keys
```
