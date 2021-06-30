# ansible-config-mgt

# Ansible Configuration Management - Automate Project 7 to 10

In Projects 7 to 10 you had to perform a lot of manual operations to set up virtual servers, install and configure required software, deploy your web application.


## Task

1. Install and configure Ansible client to act as a Jump Server/Bastion Host
2. Create a simple Ansible playbook to automate servers configuration

## Step 1 - Install and configure Ansible on EC2 Instance

1. We'll be using our Jenkins server from the previous project. 

1.  create a repo and name it ``ansible-config-mgt``
1. Instal Ansible
`sudo apt update`
`sudo apt install ansible`
1. check out ansible version by running `ansible --version`
![](https://user-images.githubusercontent.com/18899718/123687820-f965a380-d816-11eb-8983-6d9fe9806e31.png)

1. Configure Jenkins build job to save your repository content every time you change it 
- Create a new Freestyle project `ansible-mgt` in Jenkins and point it to your ‘ansible-config-mgt’ repository.
- Configure Webhook in GitHub and set webhook to trigger `ansible-mgt` build
- Configure a Post-build job to save all (**) files.

> Test your setup by making some change in master branch and make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder 
`/var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/`


![](https://user-images.githubusercontent.com/18899718/123694621-2a49d680-d81f-11eb-9349-1a2aff0832b9.png)


## Step 3 - Ansible Development
on Jenkins-Ansible server, create a folder named `ansible`
1. Change directory to `ansible` and clone repo with `git clone https://github.com/realayo/ansible-config-mgt.git`

1. Change direcotry to  `ansible-config-mgt`, create a new branch that will be used for development of a new feature. Our branch will be named `feature/prj-11-ansible`
1. Checkout the newly created feature branch
`git checkout -b feature/prj-11-ansible`
1. Create a directory and name it `playbooks`, which will be used to store all our playbook files. In the playbooks folder, create a file and name it `common.yml`
1. Create a directory and name it inventory - it will be used to keep your hosts organised. Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) `dev`, `staging`, `uat`, and `prod` respectively.

## Step 3 - Set up an Ansible Inventory

> An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. 

>Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from `Jenkins-Ansible` host - for this we have to copy our private (.pem) key to the server.
1. copy `.pem` key file from local machine to `jenkins-ansible` server. 
1. Import key to `ssh-agent` 
```
eval `ssh-agent -s`
ssh-add <path-to-private-key>
```
![](https://user-images.githubusercontent.com/18899718/123849434-698b2c80-d8de-11eb-8937-95ec3533a6fd.png)
3. Save below inventory structure in the `inventory/dev` file to start configuring our development servers. Ensure to replace the IP addresses according to your own setup:

```
[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ubuntu' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'
```


## Step 4 - Create a Common Playbook
>  `common.yml` playbook,  we will write configuration for repeatable, re-usable and multi-machine tasks that is common to systems within the infrastructure. 
1. Update your `playbooks/common.yml` file with following code:
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    yum:
      name: wireshark
      state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: ensure wireshark is at the latest version
    apt:
      name: wireshark
      state: latest
```
> The tasks was updated to create a direcotry and also change timezones on each host. 
![](https://user-images.githubusercontent.com/18899718/123853073-a3f6c880-d8e2-11eb-8d60-bbeffd25c3c2.png)

## Step 5 - Update GIT with the latest code
> We will push our update files to our remote repository.

1. in our terminal, enter `Git status` to see the latest changes 
2. `git add .` to a all the files we want to send to our remote repository
3. `git commit -m "commit message"`, this will include a description of the actions we have performed on the files we're sending to github
4. `git push` this will send our files to the remote repository.
5. On github, we'll raise a PR(Pull Request), and merge to our main branch.
![Compare and create Pull Request](https://user-images.githubusercontent.com/18899718/123859711-99403180-d8ea-11eb-809f-b608b6c1bc44.png)
![Create Pull request](https://user-images.githubusercontent.com/18899718/123859805-b8d75a00-d8ea-11eb-8d9b-a5448a002af8.png) 
![](https://user-images.githubusercontent.com/18899718/123859980-f20fca00-d8ea-11eb-8b52-b07f5d40ca47.png)
1. On the terminal, checkout from the `feature/prj-11-ansible` into the `main`, and pull down the latest changes.
![](https://user-images.githubusercontent.com/18899718/123860265-4dda5300-d8eb-11eb-9d01-01cd553c069f.png)

1. Once our code changes appear in `main` branch - Jenkins will do its job and save all the files (build artifacts) to `/var/lib/jenkins/jobs/ansible_mgt/builds/<build_number>/archive/` directory on `Jenkins-Ansible` server.

## Step 6 - Run first Ansible test
> For ansible to access the other servers, we have to generate and share ansible private key with each server.
1. on `jenkins-ansible`, edit sshd configuration
`sudo vi /etc/ssh/sshd_config` under `# Authentication:` change `PubkeyAuthentication` to `yes` also change
`PasswordAuthentication` to `yes`
2. Save and quit
3. Change password with `sudo passwd ubuntu`, this will prompt for a new password. 
4. Restart sshd with `sudo systemctl restart sshd`
5. Repeat steps 1-4 for other hosts.
6. Generate public key `ssh-keygen -t rsa`
7. Copy the generated public key to other hosts using `ssh-copy-id user@<public-ip-address>`
8. Ping all hosts to make sure ansible can acess them.
`ansible all -m ping`
![](https://user-images.githubusercontent.com/18899718/123868646-95fe7300-d8f5-11eb-94e3-9df1014c2de0.png)
9. Now, it is time to execute `ansible-playbook` command and verify if your playbook actually works
```
ansible-playbook -i /var/lib/jenkins/jobs/ansible_mgt/builds/<build-number>/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible_mgt/builds/<build-number>/archive/playbooks/common.yml
```
![](https://user-images.githubusercontent.com/18899718/123869214-52f0cf80-d8f6-11eb-9923-3cf28ab9b624.png)
10. Login to the servers and check that the insturctions given to ansible were executed.
![](https://user-images.githubusercontent.com/18899718/123869390-87fd2200-d8f6-11eb-9039-6baadd1b12f0.png)
![](https://user-images.githubusercontent.com/18899718/123869464-a6fbb400-d8f6-11eb-9d75-479ccad2426c.png)