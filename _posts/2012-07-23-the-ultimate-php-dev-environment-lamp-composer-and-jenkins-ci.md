---
layout: post
title: "The Ultimate PHP Dev Environment: LAMP, Composer, and Jenkins CI"
description: ""
category: 
tags: [LAMP, Composer, Jenkins]
---
{% include JB/setup %}
The Ultimate PHP Dev Environment: LAMP, Composer, and Jenkins CI

While attending the Lone Star PHP Conference last weekend, I learned about tools that developers are using to organize their code and keep the development process efficient. If you’re writing unit tests and using a source code versioning system, such as SVN or Git, you might want to try Jenkins CI for PHP.

Jenkins is an expandable, open-source continuous integration server. Basically, Jenkins can be scripted to perform various tasks, such as running your unit tests, sniffing your code for copy and paste or other messes, validating against custom style rules, and graphing the stability of your project. It’s incredible powerful.

Now, I am a Mac user to the core. But it seems that most prefer to run Jenkins on a Linux box. So I decided to create what I call the Ultimate PHP Dev Environment on a Linux virtual machine (VM) using Oracle’s (free) VirtualBox application. This VM will consist of:

* The BitNami Ubuntu LAMP (Linux, Apache, MySQL, PHP) Stack
* Git for the source code manager
* Composer for the PHP dependency manager
* Jenkins CI as the continuous integration server
* Sebastian Bergmann’s excellent PHP Project Wizard template for Jenkins

The advantage of using a VM is that once it’s configured, it can be cloned and saved. If you ever mess up your box, you can just clone a new one and start over without having to go through the trouble of setting everything up again. I chose the BitNami VM because it comes with a preconfigured LAMP stack that works great.

### Setting Up the VM
1. Download and install [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)
2. Download and install the [BitNami LAMP stack as a VirtualBox VM](http://bitnami.org/stack/lampstack)

Your VM should be good to go. Power up the VM and log in as the “bitnami” user, according to the instructions on the screen.

### Installing Jenkins and Composer
* Install apt-get
```bash
sudo apt-get update
sudo apt-get install
```

The following on installing Jenkins comes from [Michael Peacock](http://www.michaelpeacock.co.uk/blog/entry/jenkins-ci-an-introduction-for-php-developers):

* Get the key from the package repository
```bash
wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
```

* Edit the sources list
```bash
sudo nano /etc/apt/sources.list
```
and make it aware of the Jenkins repository, by adding the following to the bottom of the file
```bash
deb http://pkg.jenkins-ci.org/debian binary/
```
* Update the package list
```bash
sudo apt-get update
```
* Install Jenkins
```bash
sudo apt-get install jenkins
```
* Install git
```bash
sudo apt-get install git
```

Now Jenkins is installed at http://yourserverurl:8080. This should work fine, but I chose to install it at 8081 instead because other programs tend to use port 8080.

* In /etc/default/jenkins, change the HTTP_PORT value from 8080 to 8081 and save.
* Then: 
```bash
sudo service jenkins restart;
```
* Next, we set up an autoexecute file so that the SSH service is started when the VM is turned on. We also open the Jenkins port (8081) to external connections so the Jenkins web console is visible from outside the VM. Create the file /etc/init.d/autoexec.sh and put the following contents into it:
```bash
#/etc/init.d/autoexec.sh
#start SSH server and open port 8081 to external HTTP connections
service ssh start
iptables -I INPUT -p tcp --dport 8081 --syn -j ACCEPT
```
* Now do the following so the script is executed on boot:
```bash
sudo chmod +x /etc/init.d/autoexec.sh
sudo update-rc.d autoexec.sh start 99 2 . #don’t forget the “.” at the end, it’s intentional
```
* Reboot the VM. You should now be able to go to http://yourvmip:8081 from the host machine’s web browser and see the Jenkins console.
* Get Composer so it will manage your PHP project dependencies for you:
```bash
curl -s http://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```
Instructions for using Composer are below, also on the Composer site.

### Installing the PHP Project Wizard
This will provide you with a Jenkins PHP project template that you can copy and modify as needed. It’s great on it’s own, as it includes several QA tools, PHPUnit to handle your unit testing, a code sniffer, style watcher, a mess detector-- everything you need to get started.

Rather than reposting Sebastian Bergmann’s instructions here, I’m just going to [link to his page](http://sebastian-bergmann.de/archives/908-PHP-Project-Wizard.html). Follow those instructions and return here when finished. 

If you have difficulty installing phpqatools or phpDox using Sebastian’s instructions (as I did), try the following next:

```bash
sudo pear install pear.phpqatools.org/phpqatools
sudo pear channel-discover pear.phpdoc.org
sudo pear install phpdoc/phpDocumentor-alpha
sudo pear install channel://pear.php.net/Text_Highlighter-0.7.3
```

Then:

```bash
sudo pear install pear.phpqatools.org/phpqatools pear.netpirates.net/phpDox
sudo pear install channel://pear.netpirates.net/phpDox-0.4.0
```

* Restart Jenkins
```bash
sudo service jenkins restart
```

### Test it out by Creating a New Project
* Rather than coding inside the VirtualBox window, ssh into your VM from the host machine
```bash
ssh bitnami@yourvmip
```
Use the same password you used when you changed it earlier.
* Create your project directory inside the htdocs folder
```bashcd /opt/bitnami/htdocs
mkdir helloworld
```
* Create your composer.json file in the helloworld directory, according to the instructions at [getcomposer.org](http://getcomposer.org/doc/00-intro.md#declaring-dependencies). This will take care of installing all of the dependencies you list in composer.json.
* Run Composer to create your autoload file:
```bash
composer install
```
* Run the PHP Project Wizard to set up the rest of your project directory structure:
```bash
ppw --source src --tests tests --name helloworld --bootstrap vendor/autoload.php .
```
Now code your app, putting all of your source code files in the ‘src’ directory.
Learning to use Jenkins is outside the scope of this post, so refer to refer to the instructions as Sebastian’s site above.
After getting the Jenkins project template copied and applied to my project, I ran into one small problem in that Jenkins was unable to execute certain programs in PHP QA Tools:
```bash
/opt/bitnami/apache2/htdocs/helloworld/build.xml:28: 
The following error occurred while executing this line:
/opt/bitnami/apache2/htdocs/helloworld/build.xml:39: Execute failed: java.io.IOException: Cannot run program "pdepend": java.io.IOException: error=2, No such file or directory
The following error occurred while executing this line:
/opt/bitnami/apache2/htdocs/helloworld/build.xml:57: Execute failed: java.io.IOException: Cannot run program "phpcpd": java.io.IOException: error=2, No such file or directory
The following error occurred while executing this line:
/opt/bitnami/apache2/htdocs/helloworld/build.xml:69: Execute failed: java.io.IOException: Cannot run program "phpcs": java.io.IOException: error=2, No such file or directory
The following error occurred while executing this line:
/opt/bitnami/apache2/htdocs/helloworld/build.xml:78: Execute failed: java.io.IOException: Cannot run program "phpdoc": java.io.IOException: error=2, No such file or directory
The following error occurred while executing this line: 
/opt/bitnami/apache2/htdocs/helloworld/build.xml:63: Execute failed: java.io.IOException: Cannot run program "phploc": java.io.IOException: error=2, No such file or directory
```

When this happens, it is usually a permissions or PATH problem with the 'jenkins' user. Switch user to 'jenkins' using:
```bash
sudo su - jenkins
```
and add this to ~/.bash_profile:
```bash
PATH="$PATH:/opt/bitnami/php/bin"
```
Save, then reboot your VM and retry your build.

Any comments, questions, or suggestions are welcome!