
Free KMS for vSphere 6.x based on [PyKMIP](https://github.com/OpenKMIP/PyKMIP) project

### This vapp must be deployed in VCENTER environment.

During deployment, fill in IP details (address, netmask, gateway and hostname). This information can be later changed if necessary (vapp options).

Download kms4vsphere [OVA](https://app.inode.ws/index.php/s/ksm4vsphere/download/kms4vsphere-ver2.ova)

SHA256: f5f57e00bb15f591bdebffde9f8ce48835cd31b918a17f72ece49bbad0e9068d

After vapp is deployed and started, access VM via web browser (use VM IP or hostname) to download private and public certificate.

		http://ip-address-of-KMS-VM

To secure environment, after retrieving public and private key, you can remove them from internal KMS web server. Access VM console for details. Also, default user (kmip) credentials can be changed from default "Passw0rd".

To add KMS to vcenter:

1.	Highlight vcenter object in web client
2.	Go to Configure tab
3.	Go to “Key management Servers
4.	Click ADD

			* Create new KMS cluster
			* Server Name: give whatever name you want
			* Server address: IP address or hostname of KMS VM (hostname requires working DNS settings)
			* Server Port:5696
			* Save settings with ADD button

5.	To allow vcenter to trust KMS do the following:

			* Highlight just added KMS server
			* Click ESTABLISH TRUST
			* Click: Make KMS trust vCenter
			* Choose “KMS certificate and private key”
			* Paste content of public and private key in provided fields

## Reconfiguring networking inside KMS appliance

Motivation: instead of obtaining network setting from VMware Tools, use static IPv4 settings with *systemd-networkd*.

**Before you start, ensure that you have backed up KMS VM (or at least take snapshot in case you mess things up) !!!**


Switch to root account

```console
sudo -i
```


Remove netplan.io software package from the system.

```console
dpkg -P netplan.io nplan ubuntu-minimal cloud-init
```

Enable systemd-networkd service.

```console
systemctl enable systemd-networkd
```

Create */etc/systemd/network/ens160.network* file with the following content. Adjust IPv4 address, gateway and DNS to match your network. Use whatever text editor you want (vi/nano).

```
[Match]
Name=ens160

[Network]
Address=192.168.0.178/24
Gateway=192.168.0.1
DNS=192.168.0.164
```

Run these commands

```console
echo 127.0.0.1 $(hostname) localhost > /etc/hosts
systemctl unmask systemd-resolved
rm -f /etc/resolv.conf
ln -s /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

Remove old setup for network configuration.

```console
chattr -i /root/config-net.sh
rm -f /root/config-net.sh
rm -f /etc/systemd/system/config-net.service
systemctl daemon-reload
```

Create */etc/rc.local* file with the below content, and make it executable.

```bash
#!/bin/bash

# Set up DNAT to KMS container
lxc start KMS &> /dev/null

/sbin/sysctl -w net.ipv4.ip_forward=1
sleep 20

KMS_IP=$(lxc info KMS | grep -A 1 '^Ips:' | tail +2 | awk '{print $3}')
/sbin/iptables -t nat -I PREROUTING -p tcp -i ens160  --dport 5696 \
     -j DNAT --to-destination ${KMS_IP}:5696
/sbin/iptables -t nat -I PREROUTING -p tcp -i ens160  --dport 80 \
     -j DNAT --to-destination ${KMS_IP}:80

lxc file push /etc/resolv.conf KMS/etc/resolv.conf

```

```console
chmod +x /etc/rc.local
```

Reboot appliance

```console
reboot
```


