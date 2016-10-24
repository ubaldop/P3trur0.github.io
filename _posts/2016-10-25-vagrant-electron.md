---
layout: post
title: Setting up a Vagrant Machine for Electron
comments: true
spotifyTrack: spotify:track:1PJWlWKNZRZUwn1Vdhz9eh

---

Last Saturday there was an interesting conference on Node.js in Desenzano del Garda where [Feross Aboukhadijeh](http://feross.org/) held a [workshop about Electron](https://github.com/feross/electron-workshop). Since I am not enough skilled about Node.js I thought that, to attend the workshop properly, it would be worth to setup a clean VirtualBox VM to avoid any mess on my own development machine.

I am a Vagrant fan, so I’ve googled a bit trying to find a prebuilt Electron `Vagrantfile` provisioning all the packages I need. Unluckily I did not find any machine, so I had to set up it by myself.

Obviously I came across a set of small problems before to obtain a [working version](https://github.com/P3trur0/vagrant-electron) of this VM. This post describe this small journey I had to configure it.

## Preparing the provisioning scripts

#### Basic provisioning
The first stuff I've configured -as `root` user- were the basic packages required to manage a VM:


```bash
cd /home/vagrant
add-apt-repository ppa:git-core/ppa
apt-get update
apt-get install -y vim git curl
```

#### Setting up Electron

**Electron** comes as an NPM package, so I needed a clean Node.js environment to install it.
First of all, I started to address the definition of a provisioning script to setup the **Node Version Manager** used to install a working Node.js version.
Then I had to set a property to the **vm.provider** (i.e.: VirtualBox) to allow it to create _symbolic links_ in the guest machine. In fact, [as I read on the web](https://blog.liip.ch/archive/2012/07/25/vagrant-and-node-js-quick-tip.html), without this parameter NVM would not work:

```bash
config.vm.provider "virtualbox" do |vb|
  vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant","1"]
end
```

Then I had to write the provisioning script to run all the commands to setup the Node.js and Electron environment.
So I've prepared a [provisioning script](https://github.com/P3trur0/vagrant-electron/blob/master/scripts/setup.sh) setting up a bunch of packages:

```bash
  cd /home/vagrant
  echo "Installing nvm..."
  wget -qO- https://raw.github.com/creationix/nvm/master/install.sh | sh
  export NVM_DIR="/home/vagrant/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm
  nvm install node
  npm install bower -g
  npm install electron -g
```
This time, the provisioning script should run as the default `vagrant` user instead of `root`.
It:

- Downloads the latest `NVM` version
- Configures the environment variables
- Install the latest `node.js` version
- Install, globally, both `bower` and `electron` NPM packages

#### Setting up Liquidprompt

At the end of this script, I've also added the commands to setup the latest version of [Liquidprompt](https://github.com/nojhan/liquidprompt).
It is an utility to add useful information to Bash (or Zsh) prompts, based as you need it.
For example, if you're working in a Git repository, it will show you its status.

```bash
  echo "Installing liquidprompt..."
  cd /home/vagrant/
  git clone https://github.com/nojhan/liquidprompt.git
  source liquidprompt/liquidprompt
  echo "Setting up liquidprompt to the .bashrc folder"
  echo '# Only load Liquid Prompt in interactive shells, not from a script or from scp'  >> /home/vagrant/.bashrc
  echo '[[ $- = *i* ]] && source /home/vagrant/liquidprompt/liquidprompt' >> /home/vagrant/.bashrc
```

#### X Forwarding

Eventually, since Electron applications are graphical programs, it was necessary to setup the VM in order to interact with the X11 of the Host machine, otherwise it wouldn’t have been possible to see the Electron applications working.

Thanks to **X forwarding** feature, it is possible to forward graphical programs using X11 to our host machine.
To enable it, I added the following parameter to Vagrantfile, as described [here](https://coderwall.com/p/ozhfva/run-graphical-programs-within-vagrantboxes):

```bash
Vagrant.configure(2) do |config|
  ...
  config.ssh.forward_x11 = true
end
```

## Testing the VM

#### Dealing with (missing) graphical libraries

After these steps, it was time to launch the provisioning to verify if the setup was ok.
I launched the VM with the classical `vagrant up` command and I logged in it with the `vagrant ssh` command.
Once in, I've cloned the ["Hello world" Electron repository](https://github.com/electron/electron-quick-start) to verify the VM configuration:

```bash
cd /vagrant
git clone https://github.com/electron/electron-quick-start.git
cd /electron-quick-start
npm install && npm start
```

At first shot, I came across a set of problems, due to missing Ubuntu packages for graphical components.
For example I got:

```bash
error while loading shared libraries: libgtk-x11-2.0.so.0: cannot open shared object file: No such file or directory
```

Trying with repeated executions of `apt-get` commands, one by one, I got rid of all these missing libraries.
So, at the end, I had to install all the following to remove the errors:

```bash
	apt-get install -y libgtk2.0-0
	apt-get install -y libxss1
	apt-get install -y GConf2
	apt-get install -y libnss3
```

These `apt-get` commands have been added to the `root` provisioning script in order to set up the VM properly.

After all this tuning, I started the Vagrant VM and launched the **electron-quick-start** again.
This time it worked displaying, the following:

![Imgur](http://i.imgur.com/IRgDTgr.png)

Hooray!

This Vagrant VM has been tested using an Ubuntu 16.04 (64 bit) host machine. The current provider is VirtualBox (ver. 5.0.26).

_P.s.: if you need explanation of any of the listed ssh commands, please refer to [`explainshell.com`](http://explainshell.com/), I find it very useful!"_
