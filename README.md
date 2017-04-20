# Assignment 4 - DevOps

In this fourth assignment, you will be deploying your a3 application and setting up an environment that allows for you to continously update said application. 
The section [here](#assignment-walkthrough) showcases an extensive walkthrough with the dummy app contained in this repository. 

## Learning Objectives

## Table of Contents

* [Academic Integrity](#academic-integrity)
* [Assignment Walkthrough](#assignment-walkthrough)
* [Expected Functionality](#expected-functionality)
* [Extending the Assignment](#extending-the-assignment)
* [Project Submission](#project-submission)

## Academic Integrity and Collaboration

### Academic Integrity

Note that these projects should be completed **individually**.  As a result, all University-standard AI guidelines should be followed.

### Code Attribution and Collaboration

## Assignment Walkthrough
For this assignment we will be deploying our Flask app leveraging Vagrant, Ansible, Terraform, and AWS. The walkthrough is as follows:
### Vagrant Setup
For this assignment: instead of using [virtualenv](https://virtualenv.pypa.io/en/stable/) we will be using [Vagrant](https://www.vagrantup.com/). Why?

There’s two main challenges that you’ll encounter in DevOps:

* We’ll need to replicate what it’s like to set up a new server. If you’re using your local machine, it’s tough to get it to act like a fresh machine.
* It’s possible that your local machine is a different type of box than your server. Perhaps you’re working on a Mac but deploying to a Linux box, for example. Maybe you’re deploying to a different version of the operating system or a different operating system all together.

We’ll use Vagrant for our development environment because it allows us to address these challenges. As such, we will set up a virtual machine and run it there but still write code locally.

To get started we must install Vagrant [here](https://www.vagrantup.com/docs/installation/)

With vagrant installed: check success of installation with `vagrant help`
Next you will make a new directory called `vagrant` and set up a basic Vagrant configuration file that we can change to suit our needs. 
This can be done by running `vagrant init.`

```bash
$ git clone https://github.com/Cornell-PoBE/A4
$ cd A4
$ pwd
<CURR_DIRECTORY>/A4
$ mkdir vagrant
$ cd vagrant
$ vagrant init
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

Now we will customize the `Vagrantfile` for our use case. Our changes for our starter app will be the following:

* Change the config.vm.box value to ubuntu/trusty64: this sets our Virtual Machine to use one of the latest Ubuntu distributions.
* Uncomment the line that has private_network in it and note the IP address. (By default, it’s 192.168.33.10).

Your Vagrantfile should now look like this:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL
end
```

Next, in your terminal, you should run `vagrant up`. 
This will download the Ubuntu image that we specified as the config.vm.box value and create your server as a virtual machine. 
This might take a little while, depending on your connection speed. 
Once it completes, run vagrant ssh. It will put us into the command line of our virtual machine (VM).

```bash
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/trusty64'...
...
$ vagrant ssh
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-95-generic x86_64)
...
vagrant@vagrant-ubuntu-trusty-64:~$ 
```

Now in your VM you can run the following:

```bash
vagrant@vagrant-ubuntu-trusty-64:~$  sudo apt-get update
vagrant@vagrant-ubuntu-trusty-64:~$  sudo apt-get install git python-pip
...
Do you want to continue? [Y/n] Y
...
vagrant@vagrant-ubuntu-trusty-64:~$  git clone https://github.com/Cornell-PoBE/A4
vagrant@vagrant-ubuntu-trusty-64:~$  cd A4
vagrant@vagrant-ubuntu-trusty-64:~/A4$ sudo pip install -r requirements.txt
...
Successfully installed Flask gunicorn Werkzeug Jinja2 itsdangerous MarkupSafe
Cleaning up...
vagrant@vagrant-ubuntu-trusty-64:~/A4$ python app.py
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```
Now you can navigate to: `http://192.168.33.10:5000/` and see the app. 

This is a simple way of bringing our app online. But let's say we want to automate this, and in the future automate more complicated instructions and executions.

### Ansible Setup
We will be using [Ansible](https://www.ansible.com/) and its playbooks to automate the instructions above, as an example. 
Essentially, it will log in to servers that you specify using `ssh` and run commands on them. 

To begin, you will need to exit the VM we created and install Ansible on the host machine. 
```bash
vagrant@vagrant-ubuntu-trusty-64:~/A4$ exit
logout
Connection to 127.0.0.1 closed.
$ sudo pip install ansible
...
Successfully installed PyYAML-3.12 ansible-2.3.0.0 asn1crypto-0.22.0 cryptography-1.8.1 idna-2.5 ipaddress-1.0.18 packaging-16.8 paramiko-2.1.2 pycrypto-2.6.1
```
Now in your `vagrant` folder we will create a file called `site.yml`. 

This will be our Ansible [playbook](http://docs.ansible.com/ansible/playbooks.html), and it will contain our automation steps. `site` implies that this is the only file needed to get a successful version of our site up and running. The `.yml` extension tells us that it’s a YAML-formatted file (Ansible’s preference).

Your site.xml would look something like this, for our above example: 

```yml
# This is an example of a simple Ansible cookbook
---
- name: Starting a Simple Flask App
  hosts: all
  remote_user: root
  become: true
  become_method: sudo
  vars:
      repository_url: https://github.com/Cornell-PoBE/A4
      repository_path: /home/vagrant/A4

  tasks:
    - name: Install necessary packages
      apt: update_cache=yes name={{ item }} state=present
      with_items:
        - git
        - python-dev
        - python-pip
    - name: Check if A4 directory exists
      stat: path='{{ repository_path }}'
      register: a4_cloned
    - name: Pull application repo
      command: chdir='{{ repository_path }}' git pull origin master
      when: a4_cloned.stat.exists
    - name: Clone application repo
      git: repo='{{ repository_url }}' dest='{{ repository_path }}'
      when: a4_cloned.stat.exists == false
    - name: Install pip requirements
      pip: requirements='{{ repository_path }}/requirements.txt'
```

Now that we have the playbook defined, we’ll need to tell our VM to use it when setting itself up. In your Vagrantfile, uncomment the bottom-most section on provisioning and change it to look like this:

```ruby
  config.vm.provision 'ansible' do |ansible|
    ansible.playbook = 'site.yml'
    ansible.verbose = 'v'
  end
```

We’re telling Vagrant to use the `site.yml` file we created and to use verbose output. Then, let’s reprovision your host. This will run all the commands we defined in the playbook on it. `vagrant provision` should take care of it, but we will destroy the VM and restart it to prove that it works. 

```bash
$ vagrant destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Forcing shutdown of VM...
==> default: Destroying VM and associated drives...
$ vagrant up
Bringing machine 'default' up with 'virtualbox' provider...
==> default: Importing base box 'ubuntu/trusty64'...
...
==> default: Running provisioner: ansible...
    default: Running ansible-playbook...
...
$ vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ cd A4
vagrant@vagrant-ubuntu-trusty-64:~/A4$ python app.py
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

We have now built a fully automated script. For other examples your scripts will be more complex with templates, hosts, roles, etc. 

Next, we’ll need a webserver to serve requests. Serving requests through the application’s debug server poses serious security risks and it’s not intended for anything like a production load.

To serve our requests, we’re going to use `gunicorn`. Other popular options are `uWSGI` or gevent but, for the sake of constraining choice, we’ll go with this `gunicorn`.

We’re going to assume that the rest of what you’re doing here will be run within the VM unless otherwise specified.

#### Gunicorn Setup

We have already included gunicorn in our `requirements.txt` and since it is able to detect our Flask app, it is already configured! Easy!
You can run `gunicorn` with just `$ gunicorn --bind 0.0.0.0:8000 app:app`. This command will use gunicorn to serve your application through WSGI, which is the traditional way that Python webapps are served. It replaces our usual `python app.py` step. This is the simplest way to serve our application for now.

#### nginx setup

Now that we have `gunicorn` configured, we need an HTTP server to handle the requests themselves and make sure we route our users to the right application. We’ll use [nginx](https://www.nginx.com/) to do this.

As such, we will need to setup an `nginx` configuration file.
The goal of this configuration file is to make sure that we can access our server at the same host (`192.168.33.10`) but without needing to specify a port. 
Create the nginx configuration will be created in vagrant directory as well:

```bash
$ pwd
<CURR_DIRECTORY>/A4/vagrant
$ touch a4.nginx.j2
```
We will then edit `a4.nginx.j2` to contain the following:
```j2
server {
    listen 80;

    location / {
        include proxy_params;
        proxy_pass http://unix:/tmp/a4.sock;
    }
}
```
This file will tell nginx to look for our server,a unix socket, which, if our gunicorn server is running properly, should be accessible.

#### Upstart Scripts

With `gunicorn` rudimentarily configured, we’ll want to set up a script so that we can run our server automatically when our server restarts or just kick the process if it’s stuck. How we’ll do that is with an [upstart](http://upstart.ubuntu.com/) script. This script will handle starting and stopping tasks.

With an upstart scrupt we should be able to run `sudo service nginx restart` to start the task, go in your browser, and then view the service at `http://192.168.33.10`, as before. The big difference is now we have something that will run it for us, so we don’t need to SSH to run our server.

As such we will create an a4-upstart script:

```bash
$ pwd
<CURR_DIRECTORY>/A4/vagrant
$ touch upstart.conf.j2
```

We will modify that file to contain this:

```j2
description "hello-world"

start on (filesystem)
stop on runlevel [016]

respawn
setuid nobody
setgid nogroup
chdir {{ repository_path }}

exec gunicorn app:app --bind unix:/tmp/hello_world.sock --workers 3
```

However we must ensure that Ansible copies those two file into the `vagrant` directory, so each of the VMs will have access to the files. 

As such, you will modify your site.yml to be this:

```yml
# This is a simple example Ansible playbook
---
- name: Starting a Simple Flask App
  hosts: all
  remote_user: root
  become: true
  become_method: sudo
  vars:
      repository_url: https://github.com/Cornell-PoBE/A4
      repository_path: /home/vagrant/A4
  tasks:
    - name: Install necessary packages
      apt: update_cache=yes name={{ item }} state=present
      with_items:
        - git
        - python-dev
        - python-pip
        - nginx
    - name: Check if A4 directory exists
      stat: path='{{ repository_path }}'
      register: a4_cloned
    - name: Pull application repo
      command: chdir='{{ repository_path }}' git pull origin master
      when: a4_cloned.stat.exists
    - name: Clone application repo
      git: repo='{{ repository_url }}' dest='{{ repository_path }}'
      when: a4_cloned.stat.exists == false
    - name: Install pip requirements
      pip: requirements='{{ repository_path }}/requirements.txt'
    - name: Copy Upstart configuration
      template: src=upstart.conf.j2 dest=/etc/init/upstart.conf
    - name: Make sure our server is running
      service: name=upstart state=started
    - name: Copy Nginx site values
      template: src=a4.nginx.j2 dest=/etc/nginx/sites-enabled/a4
      notify:
        - restart nginx
    - name: Remove any default sites
      file: path=/etc/nginx/sites-enabled/default state=absent
      notify:
        - restart nginx
    - name: Make sure nginx is running
      service: name=nginx state=started
  handlers:
    - name: restart nginx
      service: name=nginx state=restarted
```

We’re using Ansible’s template and service modules to accomplish our task. What we’re saying is that we want to copy the template file that we defined into the directory that we used before. It will use the variables we have defined in our file, inject them into the template, and write them to the destination path we have defined.

Then, we want to make sure our service has started. If it hasn’t been started yet, start it.

The removal of `etc/nginx/sites-enabled/default` is necessary since there is a [problem](http://stackoverflow.com/questions/14972792/nginx-nginx-emerg-bind-to-80-failed-98-address-already-in-use) with the `default` site that’s enabled by `nginx`. We don’t need it, so we remove it. 

You can now run `vagrant provision`, `vagrant ssh`, and `sudo service nginx restart` to start the web server. 

```bash
$ pwd
<CURR_DIRECTORY>/A4/vagrant
$ ls
Vagrantfile     a4.nginx.j2     site.yml        upstart.conf.j2
$ vagrant provision 
==> default: Running provisioner: ansible...
    default: Running ansible-playbook...
...
TASK [Make sure nginx is running]
...
$ vagrant ssh
vagrant@vagrant-ubuntu-trusty-64:~$ cd A4
vagrant@vagrant-ubuntu-trusty-64:~/A4$ sudo service nginx restart
 * Restarting nginx nginx  						[OK]
```

You can now navigate to `http://192.168.33.10/` and see your application!

The next step in our tutorial is to deploy our application to Amazon AWS. 

#### Amazon AWS

You will be required to create an Amazon Account and launch an EC2 instance.

This EC2 instance should be using `Ubuntu Server 14.04 LTS (HVM), SSD Volume Type - ami-fce3c696` as an AMI. 
This AMI will be the same type of OS that we used for our VM.

I would recommend that you choose the `t2.micro`, which is a small, free tier-eligible instance type. 

After, launching download the the key-pair and name it `a4keypair` for the sake of consistency with this tutorial. 

Launch your instance and then ssh into system with the following command:

```bash
$ chmod 400 <CURR_DIRECTORY>a4keypair.pem
$ ssh -i <CURR_DIRECTORY>a4keypair.pem ubuntu@<SERVER_PUBLIC_IP>
```

You now have an up and running EC2 instances!

## Expected Functionality

## Extending the Assignment

## Project Submission