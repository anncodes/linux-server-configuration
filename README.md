# Linux Server Configuration Project
## Udacity Full Stack Web Developer Nanodegree
Deploying to Linux Servers Module

### About the project
Baseline installation of a Linux server and prepare it to host web applications.  Server must be secured from a number of attack vectors.  A database server will be installed and configured.  Existing web applications will then be deployed onto it.

- IP address : 35.180.79.12
- SSH port : 2200
- Application URL : [http://35.180.79.12.xip.io](http://35.180.79.12.xip.io)
- Deployed application : [Item Catalog App](https://github.com/anncodes/catalog-app)

### Get your server
The project requires [Amazon Lightsail](https://aws.amazon.com/) to create a Linux server instance.
- Create a new Ubuntu Linux server instance on Amazon Lightsail.
1. Log in.
2. Create an instance and choose an Ubuntu instance image.
   - First, choose "OS Only".Second, choose "Ubuntu" as the operating System.
3. Choose an instance plan and give your instance a hostname.
4. Once your instance has started, log into it with SSH.  Configure the Lightsail firewall.

### Configure the server
1. Update all currently installed packages.
   ```
   sudo apt-get update
   sudo apt-get upgrade
   ```
2. Change the SSH port from 22 to 2200. Configure the Lightsail firewall to allow it.
3. Configure the UFW to allow incoming connections for SSH(port 2200), HTTP(port 80), and NTP(port123).
```
WARNING: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.
```


