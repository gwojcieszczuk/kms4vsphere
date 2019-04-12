
Free KMS for vSphere 6.x based on [PyKMIP](https://github.com/OpenKMIP/PyKMIP) project

### This vapp must be deployed in VCENTER environment.

During deployment, fill in IP details (address, netmask, gateway and hostname). This information can be later changed if necessary (vapp options).

Download kms4vsphere [OVA](http://116.203.129.112/kms4vsphere/kms4vsphere-ver2.ova)

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
