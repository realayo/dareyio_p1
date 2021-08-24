# Ansible Dynamic Assignments (Include) and Community Roles

In the previous project, we did `static assignments`; in this project we will introduce `dynamic assignments` by using include module. 
```
import = Static
include = Dynamic
```
> When the import module is used, all statements are pre-processed at the time playbooks are parsed. Meaning, when you execute site.yml playbook, Ansible will process all the playbooks referenced during the time it is parsing the statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence, it is static.

> On the other hand, when include module is used, all statements are processed only during execution of the playbook. Meaning, after the statements are parsed, any changes to the statements encountered during execution will be used. 

## Step 1 - Introducing Dynamic Assignment Into Our structure

1. In our https://github.com/<your-name>/ansible-config-mgt GitHub repository create a new branch called `dynamic-assignments`.
2. Create a new folder, name it `dynamic-assignments`. Then inside this folder, create a new file and name it `env-vars.yml`. 
> Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., we will need a way to set values to variables per specific environment.
We will create a folder to keep each environment’s variables file. 

3. Create a new folder called `env-vars`, then for each environment, create new YAML files which we will use to set variables
4. Our repository folder structure should look something like this:
```
├── dynamic-assignments
│   └── env-vars.yml
├── env-vars
    └── dev.yml
    └── stage.yml
    └── uat.yml
    └── prod.yml
├── inventory
    └── dev
    └── stage
    └── uat
    └── prod
├── playbooks
    └── site.yml
└── static-assignments
    └── common.yml
    └── webservers.yml
```
5. In env-vars.yml file, paste the following code:
```
---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always
```
## Upate site.yml with dynamic assignments

In this step, we'll update `site.yml` file to make use of the dynamic assignment. 

```
---
- hosts: all
- name: this contains dynamic variables
  import_playbook: ../dynamic-assignments/env-vars.yml
 
- name: include env files
  import_playbook: ../static-assignments/common.yml
  tags:
    - always

- hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml

- hosts: db
- name: import database file
  import_playbook: ../static-assignments/db.yml
  tags:
    - always

- hosts: lb
- name: Loadbalancers assignment
  import_playbook: ../static-assignments/loadbalancers.yml
  when: load_balancer_is_required 
```

## Community Roles
Now it is time to create a role for MySQL database - it should install the MySQL package, create a database and configure users. These roles are  production ready, and dynamic to accomodate most of Linux distros.

## Step 3 — Download Mysql Ansible Role
> To preserve your your GitHub in actual state after you install a new role – make a commit and push to master your ‘ansible-config-mgt’ directory. Of course you must have git installed and configured on Jenkins-Ansible server and, for more convenient work with codes, you can configure Visual Studio Code to work with this directory
1. On `Jenkins-Ansible` server make sure that git is installed with `git --version`, in `ansible-config-mgt` directory and run
```
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```

> We will be using a MySQL role developed by geerlingguy.

1. Inside `roles` directory on the server, create your new MySQL role 
``` 
ansible-galaxy install geerlingguy.mysql 
```
2. Rename the folder to mysql
```
mv geerlingguy.mysql/ mysql
```
3. Edit roles configuration to use correct credentials for MySQL required for the tooling website. 
4. Push changes to github
```
git add .
git commit -m "Commit new role files into GitHub"
git push --set-upstream origin roles-feature
```

## Role for load Balancer
1. We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively - `nginx` and `apache`.
2. To install nginx role: 
```
ansible-galaxy install nginxinc.nginx

mv nginxinc.nginx nginx
```
3. To install apache role:
```
ansible-galaxy install geerlingguy.apache

mv geerlingguy.apache/ apache
```
4. Since you cannot use both Nginx and Apache load balancer, you need to add a condition to enable either one – this is where you can make use of variables.
5. Under `static-assignment` folder, created `loadbalancers.yml` file and copy the script below:
```
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```
6. While I updated the site.yml file with the below script:
```
 - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/lb.yml
        when: load_balancer_is_required 
```
7. In `env-vars/uat.yml`, enter the following:
```
enable_nginx_lb: true
load_balancer_is_required: true
```
8. Update `inventory/prod.yml` with the necessary server details and run the playbook.

```
ansible-playbook -i /home/ubuntu/ansible-config-mgt/inventory/prod.yml /home/ubuntu/ansible-config-mgt/playbooks/site.yml       
```
![](https://user-images.githubusercontent.com/18899718/127754624-3ba48ba6-e9ca-475c-a803-55f94b54dce6.png)
![](https://user-images.githubusercontent.com/18899718/127754637-7d7f73bc-27b4-45fd-9765-f420456a6603.png)