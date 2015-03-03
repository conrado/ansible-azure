#ansible-azure

Basic azure provisioning with ansible tutorial.

##Features

- [x] Basic HOWTO on setting up Linux controller for Azure (this file)
- [x] Ansible modules for configuring some Azure services
  - [x] ./library/azure_affinity_group
    creates and manages azure affinity groups
  - [x] ./library/azure_storage_account
    creates and manages azure storage accounts
  - [x] ./library/azure_hosted_service
    creates and manages azure hosted services
  - [x] ./library/azure_vm
    creates and manages azure virtual machines
- [ ] Scripts for configuring some Azure services
  These scripts should likely be turned into modules
  - [ ] ./bin/azure_create_vpn
    creates an azure vpn for launching machines inside

##Setting up

###Creating Azure API Management Certificates

First of all you will need access to your Azure account. Make sure you have one
and are logged into the [legacy azure management console][1] and access the
*Settings* window, then the *Management Certificates* tab.

[1]: https://manage.windowsazure.com

You will need to create a key, and then a certificate to upload to Azure.

to create the .pem, basically the master key, use the following command

```
  openssl req \
    -x509 -nodes -days 365 \
    -newkey rsa:2048 -keyout mycert.pem -out mycert.pem
```

To create the .pfx, which you will need for provisioning VMs, use the following
command, be sure not to add a password (hit enter twice)

```
  openssl pkcs12 -export \
    -out mycert.pfx -in mycert.pem \
    -name “My Certificate”
```

to create the .cer, to upload into the console so you can manage via the API,
use the following command

```
  openssl x509 -inform pem -in mycert.pem -outform der -out mycert.cer
```

Note that Azure web interface *requires* that the certificate be suffixed with
.cer, or it will not accept it as valid file xD

Place these files in the `./certs/` directory for safe keeping. This repository
will ignore that directory, but refer to the files in it for simplicity.

Now that we have a certificate we need to upload it to the management console,
while on the *Management Certificates* tab, click on the `Upload` button and add
your certificate.

###Configuring Azure modules variables for accessing API

Now that we have a certificate uploaded we can use it to configure access to
the Azure management API on this mini ansible distribution.

Once again on the *Management Certificates* tab, take note of the *Subscription ID*
which we will need to feed to the azure python SDK. It should look something
like this: `04ca5a03-2e0c-4c76-b267-5c10162d3c83`

Configure this ID under the `subscription_id` on `./vars/azure.yml`

While there, you may configure `certificate_path` if you did not place the `.pem`
in the `./certs/` directory with the standard name.

Still in `./vars/azure.yml` be sure to set the correct `fingerprint`, which
you may retrieve with the following command, then lowercase it, and remove the
colons.

```
  openssl pkcs12 \
    -in mycert.pfx -nodes -passin pass:”” | openssl x509 -noout -fingerprint
```

Should generate something like:

```
  MAC verified OK
  SHA1 Fingerprint=C5:04:49:04:82:78:EA:F1:6E:13:47:22:57:1C:85:F3:BB:3F:86:41
```

Turn this into something like:

```
  c50449048278eaf16e134722571c85f3bb3f8641
```

Still in `./vars/azure.yml` be sure the `sshcert` variable is pointing to
the correct file, default is `./certs/mycert.pfx`

###Creating a VPN

As of this writing, creating a VPN on Azure is not automated so you must create
one manually. To do so access the *Networks* item on the Azure console menu and
enter the wizard. Call your network `VNDevops`. You may use the free Google DNS
`8.8.8.8`. Make it a `point-to-site` VPN. And use the standard settings of
`10.0.0.0/24` for address space and starting ip. On Virtual Network Address
Spaces, you will create a subnet with the standard `Subnet-1` name, with
standard starting ip `10.0.1.0`, and add a gateway subnet. Click okay.

###All set up!

By now you should be set up to run the ansible scripts to automatically
provision virtual machines, storage accounts, and so forth, on your Azure account,
for the specific VPN project `VNDevops`. It is recommended you put all your
the machines that belong in a specific platform under the same VPN. So if you have
another platform you are building.. You will should create another VPN. These
more advanced topics go beyond the scope of this document.

##Provisioning virtual machines automatically.

**Vroom! Vroom! Vroooom! Fire ze missiles!!!**

Now that we are all set up, provisioning virtual machines is very easy. The ansible
playbook `./tutorial.yml` will take the following steps:

- Create an Affinity Group

  This is akin to an AWS security group. It allows you to associate machines with
  one another.

- Create a Storage Account

  This is sort of like a AWS s3. You will be able to put data into buckets here.

- Create an Hosted Service

  ??

- Create a VM

  Of course this is like creating an AWS EC2 instance.
