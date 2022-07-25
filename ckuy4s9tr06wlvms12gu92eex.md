## Enjoy Infomaniak IaaS Public Cloud with Laravel Forge

Infomaniak released a new service on September 28, 2021: **[Public Cloud](https://news.infomaniak.com/en/public-cloud-alternative-gafam)**, an IaaS based on Debian and OpenStack . It's the infrastructure they already use for their own services, hosted in Switzerland and committed to a sustainable economy and customer privacy (ISO 14001, ISO 50001, ISO 27001). 

**[Laravel Forge](https://forge.laravel.com)** is a server management solution for easy provisioning, management and deployment of PHP applications (push to deploy, accessible configuration, database backup, SSL certificates, ...) provided by the Laravel squad.

As Laravel Forge doesn't support Infomaniak as a provider (for now), there are some additional steps to make it work together.

We assume you already have an account on Laravel Forge, Infomaniak and that you [created a project on the Public Cloud service](https://www.infomaniak.com/en/support/faq/2603/public-cloud-create-a-new-project).

Here are the steps to have a running and managed server:
- [Infomaniak Public Cloud](#infomaniak-public-cloud)
  - [Create and launch a new Instance](#create-and-launch-your-instance-in-infomaniak-public-cloud)
  - [Add SSH and Web access to your Instance](#add-ssh-and-web-access-to-your-instance)
- [Laravel Forge](#laravel-forge)
  - [Create the Server](#create-the-server)
  - [Provision it](#provision-it)
- [🙌](#8jzja)

## Infomaniak Public Cloud

### Create and launch your Instance in Infomaniak Public Cloud

Once you are on your project page, go to **Compute > Instances** menu and click on **Launch Instance** button.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634546631019/6nMkv3s-K.png)

You now have a modal window requiring information you have to fill in multiple steps:

#### Details (basic information)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634546662234/dfATzOIxQ.png)

Just give your Instance a name (and a description if you want). You don't need to set an availability zone.

#### Source (OS to install)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634546711651/4Xi_qmHKB.png)

Laravel Forge requires a **Ubuntu 20.04 x64** or **Ubuntu 22.04 x64**, so simply add **Ubuntu 22.04 LTS** from the list if you want the latest release.

#### Flavor (ressources of the Instance)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634546763417/Tc4V1hpFk.png)

Here you have to choose the flavor of your Instance (CPU, RAM, storage).
You can find the pricing for each Instance flavor here:  [https://www.infomaniak.com/en/hosting/public-cloud/prices](https://www.infomaniak.com/en/hosting/public-cloud/prices) 

#### Networks

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634546855262/n1cJLHL6m.png)

Select the provided Infomaniak IPV4 external network **ext-net1**.

#### Network Ports

Nothing specific to do here.

#### Security Groups

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634638399592/-i-8DDeU5.png)

Here you define the opened ports (entering and exiting) and authorized IP addresses of your Instance, using Security Groups feature.

Keep **default**, for now you will create and add **ssh** (to allow Laravel Forge to manage your Instance), **http** and **https** (to make your Web application reachable) access afterwards.

#### Key Pair

If you didn't add a SSH key previously, you can import it here.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634547055961/9MlzehbT1.png)

You can now associate it to your Instance.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634547099587/HY7R3TkNB.png)

#### Configuration

Nothing specific to do here.

#### Server Groups

Nothing specific to do here.

#### Scheduler Hints

Nothing specific to do here.

#### Metadata

Nothing specific to do here.

#### Launch the Instance

All is now set, you can launch your Instance

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634547213090/BA6Rc175L.png)

and see it running a few seconds later.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634547371730/KHYTtuRfW.png)

### Add SSH and Web access to your Instance

On a fresh Infomaniak Public Cloud project, there's no Security Groups created to allow SSH and Web access, you have to create them (you will be able to reuse them on the other Instances you will create in your project afterwards).

To do so, go to **Network > Security Groups** menu and click on **Create Security Group** button.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634639531441/wYWxa4clt.png)

#### SSH

On the opening modal window, give your Security Group a name (and a description if you want) and click on **Create Security Group** button.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634549039804/qiMZ8GAmY.png)

By default, the Security Group gives exiting (Egress) access to any IP on any port. You don't need those rules as they already are set in the **default** Security Group, you can then delete them.

You can now add the Rule you need, so click on **Add Rule** button.

On the opening modal window, you can see a predefined list of protocols, select **SSH**. You can also filter the allowed IP addresses (for Laravel Forge and your own IP addresses for example, you will allow everybody for the moment).

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634549079576/GUMtDgq4_.png)

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634549121350/xyiENPmpj.png)

#### Web

Follow the same steps as for SSH for the Security Group creation and default Rules deletion.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634548953991/Fjotr-Jl0.png)

Then, for this Security Group, you will add 2 Rules (that are also predefined):
- HTTP
- HTTPS

As you want everybody to have access to our Web application, you don't filter IP addresses.

#### Add the Security Groups to your Instance

You can now go back to the **Compute > Instances** menu, and in the **Actions** column of your Instance, click on **Edit Security Groups**

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634547983207/M0ycuhMg8K.png)

You can now add your **ssh** and **web** Security Groups:

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1634641007699/BUB2BNTVW.png)

Once saved, the rules are instantly applied. All is done now for the Infomaniak Public Cloud part (and it will be quicker for the next Instances, as you'll be able to add the **ssh** and **web** Security Groups directly on Instance creation).


## Laravel Forge

### Create the Server

On the Server creation section, select **Custom VPS**.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643301679924/z1oxciKN5.png)

On the form that appears:
- Select the type of the Server: App Server (including all the softwares needed for a Web application), or you can also split your application in multiple Servers / Instances
- Give your Server a name (maybe the same as your Infomaniak Public Cloud Instance is a good idea?)
- Set the IP address of the Server (that you can find in the Infomaniak Instances list)
- Change the PHP version and Database type if you want
- Click on **Create Server** button.

### Provision it

You will now see a modal window where you can find:
- The script provided by Laravel Forge to set up your Server and make it fully managed
- The passwords for sudo and the Database (if you choose a server type that includes a Database).

> Don't click on Close too quickly and keep those information somewhere secure.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643301835894/a9l3oRdz2.png)

For a supported Provider, Laravel Forge runs the script automatically without requiring any further action from you. But with Custom Providers, it is to us to run it manually, so:

- SSH into your Instance (or Server, or whatever you want to call it)
```sh
ssh ubuntu@195.15.xxx.xxx
```
- Use sudo to be root
```sh
sudo -s
root@my-server:/home/ubuntu#
```
- Run the provisioning command
```sh
wget -O forge.sh "https://forge.laravel.com/servers/XXXXXX/vps?forge_token=xxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXXxxxxxXXxxXXXXxxxxXXXxxxXXXxxxXXXXXX&recipe="; 
bash forge.sh
```

If you go back to Laravel Forge and look at your Server, you should now see it being prepared.

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643301951903/yMVwyRJoF.png)

## 🙌

Here you are, the Instance is running, installed, configured, managed and the default Laravel Forge site is deployed, you can open a browser and go to your Instance IP address to confirm

![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643302586445/oTObg92qT0.png)

And of course, you can now deploy your PHP application and enjoy the Infomaniak infrastructure and Laravel Forge quick deploy feature!






