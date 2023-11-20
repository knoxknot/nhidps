#### USAGE
This README.md documents installation and running of a Network and Host Instrusion Dectection and Prevention System.  

Install RedHat Ansible, Oracle Virtualbox, HashiCorp Packer and Vagrant binaries.
```shell
# install RedHat Ansible
sudo apt install ansible -y

# install Oracle Virtualbox
Arch=amd64; Distro=Debian; Distro_Codename=bookworm; Version_Build_Number=7.0.12-159484; Version_Major_Number=7.0; Version_Minor_Number=7.0.12
wget --secure-protocol TLSv1_3 "https://download.virtualbox.org/virtualbox/${Version_Major_Number}/virtualbox-${Version_Build_Number}~${Distro}~${Distro_Codename}_${Arch}.deb"  # download virtualbox
wget --secure-protocol TLSv1_3 https://download.virtualbox.org/virtualbox/${Version_Major_Number}/Oracle_VM_VirtualBox_Extension_Pack-${Version_Major_Number}.vbox-extpack  # download virtualbox extension
wget --secure-protocol TLSv1_3  https://www.virtualbox.org/download/hashes/${Version_Major_Number}/SHA256SUMS
grep "${Distro_Codename}" SHA256SUMS | sha256sum -c  # verify integrity

sudo apt install -f ./virtualbox-${Version_Major_Number}_${Version_Build_Number}~${Distro}~${Distro_Codename}_${Arch}.deb  # install virtualbox
sudo VBoxManage extpack install Oracle_VM_VirtualBox_Extension_Pack-${Version_Minor_Number}.vbox-extpack  # install virtualbox extension
sudo usermod -aG vboxusers $USER  # add current user to vboxusers group

# install Hashicorp Packer
Arch=amd64; OS=linux; Version=1.9.4  # specify variables for download
wget --secure-protocol TLSv1_3 https://releases.hashicorp.com/packer/${Version}/packer_${Version}_${OS}_${Arch}.zip  # download packer binary archive
unzip -l packer_${Version}_${OS}_${Arch}.zip  # view archive content
sudo unzip packer_${Version}_${OS}_${Arch}.zip -d /usr/local/bin/ packer  # unarchive and install binary

# install Hashicorp Vagrant
Arch=amd64; OS=linux; Version=2.4.0  # specify variables for download
wget --secure-protocol TLSv1_3 https://releases.hashicorp.com/vagrant/${Version}/vagrant_${Version}_${OS}_${Arch}.zip  # download vagrant binary archive
unzip -l vagrant_${Version}_${OS}_${Arch}.zip  # view archive content
sudo unzip vagrant_${Version}_${OS}_${Arch}.zip -d /usr/local/bin/ vagrant  # unarchive and install binary
```

Download and verify the ISO image 
```shell
# download iso image
Arch=amd64; Desktop=xfce; Version=12.2.0
wget --secure-protocol TLSv1_3 https://cdimage.debian.org/debian-cd/current-live/${Arch}/iso-hybrid/debian-live-${Version}-${Arch}-${Desktop}.iso -P ~/Software/

# download iso checksum
wget --secure-protocol TLSv1_3 https://cdimage.debian.org/debian-cd/current-live/${Arch}/iso-hybrid/SHA256SUMS -O ~/Software/debian-${Version}-SHA256SUMS 

# verify iso integrity
grep -E "debian-live-${Version}-${Arch}-${Desktop}.iso$" debian-${Version}-SHA256SUMS | sha256sum -c  
```

Bootstrap, build and run the Network and Host Instrusion and Prevention System.
```shell
# hash the passwords and insert in the preseed file
ROOT_PASSWORD="INSERT_ROOT_PASSWORD_HERE"; USER_PASSWORD="INSERT_USER_PASSWORD_HERE"
echo $ROOT_PASSWORD | mkpasswd -s -m sha-512  # replace passwd/root-password-crypted value
echo $USER_PASSWORD | mkpasswd -s -m sha-512  # replace passwd/user-password-crypted value

# building the system with packer json definition
packer plugins install github.com/hashicorp/virtualbox
ISO_URL="INSERT_ISO_FILEPATH" TMPDIR=./ PACKER_LOG_PATH=packer.log PACKER_LOG=2 packer build -var-file template-vars.json -force template.json

# running the system
ssh-keygen -t ed25519 -b 4096 -f nhidps -C "nhidps keypair" -N ""  # create an ssh key
vagrant up  # start the vm
```

NHIDPS git workflow
```shell
gh repo create nhidps --public --homepage https://github.com/knoxknot --description "Building a Network/Host Instrusion/Prevention System"  # programmatically create a remote repository
git init .  # initialize git in the current directory
git branch -M main  # rename the default master branch to main
git commit --allow-empty -m "initial commit"  # initial empty commit
git remote add origin https://github.com/knoxknot/nhidps.git  # point local to remote
git push origin main  # push to remote main
git checkout -b develop  # create and switch to develop branch
git status  # list untracked resources
git add .  # add untracked resources to staging area
git commit -m "chore: developed automated scripts for debian fresh install"  # commit changes
git push -u origin develop  # push specified branch local to remote
```