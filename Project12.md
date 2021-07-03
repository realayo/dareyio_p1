# Ansible Refactoring & Static Assignments

`It is important to refactor Ansible codes, create assignments, and learn how to use the imports functionality. Imports allow us to effectively re-use previously created playbooks in a new playbook which lets us organise tasks and reuse them as needed.`

## Step 1 - Jenkins job enhancement

> We need to make some modifications to our Jenkins job. Because Jenkins creates a new directory each time a change is triggered, running a command can be very challenging and a lot of folders are created which can take up unnecessary space on the server. 

> We can mitigate this by using `Copy Artifact plugin`.

1. On `Jenkins-Ansible` server, create a new directory called `ansible-config-artifact` which will store all artifacts after each build.
```
sudo mkdir /home/ubuntu/ansible-config-artifact
```
2. Change permissions to this directory, so Jenkins can save files there
```
chmod -R 0777 /home/ubuntu/ansible-config-artifact
```
3. On `Jenkins web console` -> `Manage Jenkins` -> `Manage Plugins` -> on `Available` tab search for `Copy Artifact` and install this plugin without restarting Jenkins
4. Create a new `Freestyle project` and name it `save_artifacts`, this project will be triggered by completion of our existing `ansible_mgt` project. Configure it accordingly:
![](https://user-images.githubusercontent.com/18899718/124343059-48e8fe00-db8e-11eb-9091-6d39e260e997.png)
![](https://user-images.githubusercontent.com/18899718/124343094-83529b00-db8e-11eb-92a5-d3af94b5b1e6.png)

>  You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results.
5. The main idea of `save_artifacts` project is to save artifacts into `/home/ubuntu/ansible-config-artifact` directory. To achieve this, create a `Build`step and choose `Copy artifacts from other project`, specify `ansible_mgt` as a source project and `/home/ubuntu/ansible-config-artifact` as a `target directory`.
![](https://user-images.githubusercontent.com/18899718/124343477-52279a00-db91-11eb-83a0-c220882999f4.png)

6. Test your set up by making some change in README.MD file inside `ansible-config-mgt repository`(main branch) 
![](https://user-images.githubusercontent.com/18899718/124343535-ce21e200-db91-11eb-951f-87545a81d984.png)

## Step 2 - Refactor Ansible code by importing other playbooks into `site.yml`

> Here, we're breaking tasks up into different files so as to organise sets of tasks and reuse them.
Now, we'll try code re-use by importing other playbooks.

> checkout to a new branch called `refactor` 
```
git checkout -b refactor
```
1. Within `playbooks folder`, create a new file and name it `site.yml` which would be an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. which means `site.yml` will become a parent to all other playbooks that will be developed.
2. Create a new folder in root of the repository and name it `static-assignments`. The `static-assignments` folder is where all other children playbooks will be stored. This is merely for easy organisation of our work. 
3. Move `common.yml` file into the newly created `static-assignments` folder.
4. Inside `site.yml` file, import `common.yml` playbook.
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml
```
Our folder structure should look like this:
```
├── static-assignments
│   └── common.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
```

4. Run `ansible-playbook` command against the `dev` environment. 
> We would need to apply some tasks to our `dev` servers. Since wireshark is already installed, we can create another playbook under `static-assignments` and name it common-del.yml. In this playbook, we would configure deletion of wireshark utility.
```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
```
5. update site.yml` with
```
import_playbook: ../static-assignments/common-del.yml
```
instead of common.yml and run it against `dev` servers.
```
ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/dev.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml
```
![](https://user-images.githubusercontent.com/18899718/124343937-ffe87800-db94-11eb-9fbd-8f9db8d9fc0e.png)
6. Make sure that `wireshark` is deleted on all the servers by running `wireshark --version`
![](https://user-images.githubusercontent.com/18899718/124343964-40e08c80-db95-11eb-88ba-7aeebde92eb3.png)
![](https://user-images.githubusercontent.com/18899718/124343969-4f2ea880-db95-11eb-83cd-ddb8b102b334.png)

## Step 3 - Configure UAT Webservers with a role ‘Webserver’

> We are going to configure 2 new Web Servers as `uat`. We could write tasks to configure Web Servers in the same playbook - `site.yml`, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.
1. Launch 2 EC2 instances using `RHEL 8` image, which would serve as our `uat` servers. We would name  `Web1-UAT` and `Web2-UAT`
2. to create a role, we must create a `roles` directory. If you have ansible installed on your local machine, do:
```
mkdir roles
cd roles
ansible-galaxy init webserver
```
3. If you don't have ansible installed, just create your folders in this structure:
```
└── webserver
    ├── README.md
    ├── defaults
    │   └── main.yml
    ├── handlers
    │   └── main.yml
    ├── meta
    │   └── main.yml
    ├── tasks
    │   └── main.yml
    └── templates
```
4. Update `uat` inventory file with IP addresses of our 2 `uat` Web servers
```
[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' ansible_ssh_private_key_file=<path-to-.pem-private-key>
```
![](https://user-images.githubusercontent.com/18899718/124344568-9cf8e000-db98-11eb-9fe9-a58890b3a526.png)
5. In `/etc/ansible/ansible.cfg` file uncomment `roles_path` string and provide a full path to our roles directory `roles_path = /home/ubuntu/ansible-config-mgt/roles`, so Ansible could know where to find configured roles.
```
sudo vi /etc/ansible/ansible.cfg

# additional paths to search for roles in, colon separated
#roles_path    = /etc/ansible/role
roles_path    = /home/ubuntu/ansible-config-mgt/roles
```
> Now, we'll start adding some logic to the `webserver role`. 
6. Go into `tasks` directory, and within the `main.yml` file, we'll write configuration tasks to do the following:
* Install and configure Apache (httpd service)
* Clone Tooling website from GitHub https://github.com/<your-name>/tooling.git.
* Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
* Make sure httpd service is started
```
--
# tasks file for weberver

- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/realayo/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

## Step 4 - Reference ‘Webserver’ role
1. Within the `static-assignments `folder, create a new assignment for `uat-webservers` in `uat-webservers.yml`. This is where you will reference the role.
```
---
- hosts: uat-webservers
  roles:
     - webserver
```
2. Since the entry point to our ansible configuration is `site.yml` file. We need to call our `uat-webservers.yml` role inside `site.yml`.
```
---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
```

## Step 5 - Commit & Test
1. Commit your changes to the `refactor branch` and push to GitHub, create a pull request and merge with the main branch. The artifacts should be automatically copied to the `ansible-config-artifact` directory.
2. Test connection
![](https://user-images.githubusercontent.com/18899718/124345643-d719b000-db9f-11eb-9cfc-672e19a88672.png)
2. Run the playbook against `uat` inventory
```
sudo ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/uat.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yaml
```
![](https://user-images.githubusercontent.com/18899718/124345191-1b578100-db9d-11eb-99c8-5b2e3260acac.png)
4. UAT Web servers are now configured and can be reached from any browser. 

* http://Web1-UAT-Server-Public-IP-or-Public-DNS-Name/index.php
![](https://user-images.githubusercontent.com/18899718/124345237-5ce82c00-db9d-11eb-839b-5c1b0453cb6c.png)
http://Web1-UAT-Server-Public-IP-or-Public-DNS-Name/index.php
![](https://user-images.githubusercontent.com/18899718/124345256-6ec9cf00-db9d-11eb-9458-dfdb66570468.png)