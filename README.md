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

Import the public key for ELRepo:

```
sudo rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
```

Install ELRepo for RHEL-8 or CentOS-8:

```
sudo yum install https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
```

Install ELRepo for RHEL-7, SL-7 or CentOS-7:

```
sudo yum install https://www.elrepo.org/elrepo-release-7.el7.elrepo.noarch.rpm
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
module_hotfixes=true

[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
```

By default, the repository for stable nginx packages is used. If you would like to use mainline nginx packages, run the following command:

```
sudo yum-config-manager --enable nginx-mainline
```

To install nginx, run the following command:

```
sudo yum install nginx
```

When prompted to accept the GPG key, verify that the fingerprint matches `573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62`, and if so, accept it.

Command to check nginx configurations:

```
sudo nginx -t
```

Command to enable and start nginx:

```
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Remi's RPM Repository

We would install  `php80` in  `multiple versions simultaneously` mode in CentOS 8.

For more information, please refer to the [Configuration wizard](https://rpms.remirepo.net/wizard/).

Command to install the Remi repository configuration package:

```
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

Command to install the php80 collection:

```
sudo dnf install php80
```

Command to install additional packages:

```
sudo dnf install php80-php-fpm
```

Command to check the installed version and available extensions:

```
php80 --version
php80 --modules
```

Command to enable and start php74:

```
sudo systemctl enable php80-php-fpm
sudo systemctl start php80-php-fpm
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

### SELinux and Permission Settings

To get the status of a system running SELinux:

```
sestatus
```

The output should indicate SELinux enabled and enforcing on default CentOS installation:

```
SELinux status:                 enabled
SELinuxfs mount:                /sys/fs/selinux
SELinux root directory:         /etc/selinux
Loaded policy name:             targeted
Current mode:                   enforcing
Mode from config file:          enforcing
Policy MLS status:              enabled
Policy deny_unknown status:     allowed
Max kernel policy version:      31
```

To display security context of a file or folder:

```
ls -Z
```

Change file SELinux security context to  `Read-only directories and files used by php-fpm`:

```
sudo chcon -Rv --type="httpd_sys_content_t" www/
```

Change file SELinux security context to  `Readable and writable directories and files used by php-fpm`:

```
sudo chcon -Rv --type="httpd_sys_rw_content_t" www/
```

Change any new nginx site configuation file SELinux security context to  `httpd_config_t` and user in security context to  `system_u`:

```
sudo chcon -v --user="system_u" --type="httpd_config_t" /etc/nginx/conf.d/example.conf
```

Change the web folder owner to  `apache:apache`:

```
sudo chown -R apache:apache example/
```

Allow web content to access network resources:

```
setsebool -P httpd_can_network_connect=1
```

Allow web content (e.g. Adminer) to access database on the network:

```
setsebool -P httpd_can_network_connect_db=1
```

### dhparam Generation

The generation process would take a few minutes to complete.

```
sudo openssl dhparam -out /etc/pki/tls/private/dhparam.pem 4096 && sudo chmod 0640 /etc/pki/tls/private/dhparam.pem
```
