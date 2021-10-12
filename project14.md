
# Continuous Integration with Jenkins | Ansible | Artifactory | SonarQube | PHP

## Step 1: Simulating a typical CI/CD Pipeline for a PHP Based application

This is a continuation of Project 11 through Project 13.
Note: Create servers required for an environment you are working with at the moment only. For example, when doing deployments for development, do not create servers for integration, pentest, or production yet.

## Step 1.1: Set Up
Ansible inventory should look like this:
```
├── ci
├── dev
├── pentest
├── preprod
├── prod
├── sit
└── uat
```

ci inventory
```
[jenkins]
<Jenkins-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[sonarqube]
<SonarQube-Private-IP-Address>

[artifact_repository]
<Artifact_repository-Private-IP-Address>
```

dev
```
[tooling]
<Tooling-Web-Server-Private-IP-Address>

[todo]
<Todo-Web-Server-Private-IP-Address>

[nginx]
<Nginx-Private-IP-Address>

[db:vars]
ansible_user=ec2-user
ansible_python_interpreter=/usr/bin/python

[db]
<DB-Server-Private-IP-Address>
```

pentest
```
[pentest:children]
pentest-todo
pentest-tooling

[pentest-todo]
<Pentest-for-Todo-Private-IP-Address>

[pentest-tooling]
<Pentest-for-Tooling-Private-IP-Address>
```

## Step 1.2: Add two more roles to your Ansible playbook

    Sonarqube: Sonarqube is an open-source platform developed by SonarSource for continuous inspection of code quality to perform automatic reviews with static analysis of code to detect bugs, code smells, and security vulnerabilities.

    Artifactory: JFrog Artifactory is a repository for all software package types. Store build artifacts in an Artifactory repository. JFrog Artifactory is a repository manager that supports all available software package types, enabling automated continous integration and delivery. 

## Step 1.3: Configure Ansible for Jenkins development

In previous projects, you have been launching Ansible commands manually from a CLI. Now, with Jenkins, we will start running Ansible from Jenkins UI.

1.  Navigate to Jenkins UI and install Blue Ocean plugin

2. Click Open Blue Ocean from the left pane

3. Once you're in the Blue Ocean UI, click Create Pipeline

4. Select GitHub

5. On the Connect to GitHub step, click 'Create an access token here' to create your access token

6. Type in the token name of your choice, leave everything as is and click Generate token

7. Copy the generated token and paste in the field provided in Blue Ocean and click connect

8. Select your organization (typically your GitHub repository name)

9. Select the repository to create the pipeline from

10. Blue Ocean would take you to where to create a pipeline, since we are not doing this now, click Administration from the top bar to exit Blue Ocean.

### Create Jenkinsfile
1. Inside the Ansible project, create a deploy folder
2. In the deploy folder, create a file named Jenkinsfile
3. Add the following snippet into the newly created file named Jenkinsfile
```
pipeline {
  agent any
    stages {
      stage('Build') {
        steps {
          script {
            sh 'echo "Building Stage"'
          }
        }
      }
    }   
}
```
> The above pipeline job has only one stage (Build) and the stage contains only one step which runs a shell script to echo "Building stage"

1. Go back to Ansible project in Jenkins and click `Configure`

2. Scroll down to `Build Configuration` and for script path, enter the path to the Jenkinsfile (`deploy/Jenkinsfile` in our case)

3. Go back to the pipeline and click `Build Now`

4. Open `Blue Ocean` again to see the build in action

You could trigger the build again by clicking the play button.

5. Since our pipeline is multibranch, we could build all the branches in the repo independently. To see this in action,

6. Create a new git branch and name it `features/jenkinspipeline-stages`

7. Add a new build stage "Test"
```
pipeline {
  agent any
    stages {
      stage('Build') {
        steps {
          script {
            sh 'echo "Building Stage"'
          }
        }
      }

        stage('Test') {
          steps {
            script {
              sh 'echo "Testing Stage"'
              }
          }
        }
    }
}
```
8. To make the new branch show in Jenkins UI, click `Administration` to exit Blue Ocean, click the project and click `Scan Repository Now` from the left pane.

9. Refresh the page and you should see the new branch.

10. Open `Blue Ocean` and you should see the new branch.

![](https://user-images.githubusercontent.com/18899718/136045395-a9e796a8-ec5b-4a20-aa7b-e28de80ab907.png)

Quick task:
1. Create a pull request to merge the latest code into the `main branch`
2. After merging the `PR`, go back into your terminal and switch into the `main` branch.
3. Pull the latest change.
4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an `echo` command like we have in `build` and `test` stages)
    1. Package 
    2. Deploy 
    3. Clean up
5. Verify in Blue Ocean that all the stages are working, then merge the feature branch to the main branch
6. Eventually, the main branch should have a successful pipeline like this in Blue Ocean

Your final pipeline should look like this: ![](https://user-images.githubusercontent.com/18899718/136047005-8dd0030a-38c0-4df1-a165-082b6711650f.png)

## Step 1.4: Running Ansible Playbook from Jenkins

1. Install Ansible on your Jenkins server
```
sudo apt install ansible -y
```
### Install Ansible plugin in Jenkins UI
1. Go to `Manage Jenkins`
2. Click 1Manage Plugins`
3. Click `Available` tab and in the search bar, enter `Ansible` and click the check box next to the plugin
4. Scroll down and click `Install without restart`
5. Once ansible plugin has been installed, go to `Manage Jenkins`
6. Click on `Global Configuration Tools`
7. Scroll down to `Ansible` and click on it, in the `Name` box, type `ansible` and in the `Path to ansible Executables directory` box, type the path to ansible(you can check this using command `which ansible` on the Linux terminal), in our case, it's `/usr/bin`
8. Create Jenkinsfile from scratch (delete all the current stages in the file)

9. Add a new stage to clone the GitHub repository

```
stage('SCM Checkout') {
  steps {
            git(branch: 'main', url: 'https://github.com/TheCountt/config-ansible-mgt.git')
          }
        }
```
Add the next stage, to run the playbook. For this, we need to click on Pipeline Syntax at bottom left of Jenkins UI and input appropriate value and generate a pipeline script. You can watch this [video](https://www.youtube.com/watch?v=PRpEbFZi7nI&feature=youtu.be) as a guide.

```
stage('Execute Ansible') {
    steps {
        ansiblePlaybook colorized: true, credentialsId: 'privateKEY', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory_file}', playbook: 'playbooks/site.yml'}
    }
  }
}
```
This build stage requires a credentails file - private key - which can be created by following these steps:
1. Click `Manage Jenkins` and scroll down a bit to `Manage Credentials`
2. Under `Stores` go to `Jenkins` on the right, click on Jenkins
3. Under System, click the `Global credentials (unrestricted)`
4. Click `Add Credentials` on the left
5. For `credentials kind`, choose `SSH username with private key`
6. Enter the ID as `private-key`
Enter the username Jenkins would use (ubuntu/ec2-user)
7. Enter the `secret key` (the contents of your `privatekey.pem` file from AWS or any cloud provider)
8. Click `OK` to save

>You can add a stage that cleans your workspace after every build

```
stage('Clean up') {
  steps {
    cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
  }
}
```
9. Commit and push your changes

10. Go to Blue Ocean and trigger a build against the branch. If everything is configured properly, you should see something like this:
![](https://user-images.githubusercontent.com/18899718/136063078-9854761c-f0f5-41a7-8759-36221b45bdf7.png)


## Parameterizing Jenkinsfile for Ansible Development

To deploy to other environments, we have to use parameters

1. Update SIT environment inventory file with new servers

Update Jenkinsfile to add parameters
```
pipeline{
 agent any
 environment {
    ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
  }
   parameters {
    string(name: 'inventory_file', defaultValue: '${inventory_file}', description: 'selecting the environment')
   }
}
```
    
Overall, the Jenkinsfile in the deploy folder(deploy/Jenkinsfile) should like this

```
pipeline{
 agent any
 environment {
    ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
  }
   parameters {
    string(name: 'inventory_file', defaultValue: '${inventory_file}', description: 'selecting the environment')
        }
 stages{
    stage("Initial cleanup") {
        steps {
          dir("${WORKSPACE}") {
            deleteDir()
          }
        }
      }
    stage('SCM Checkout') {
       steps{
          git branch: 'main', url: 'https://github.com/TheCountt/config-mgt-ansible.git'
       }
     }
    stage('Prepare Ansible For Execution') {
      steps {
        sh 'echo ${WORKSPACE}'
      }
   }
    stage('Execute Ansible Playbook') {
      steps {
          ansiblePlaybook colorized: true, credentialsId: 'privateKEY', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory_file}', playbook: 'playbooks/site.yml', skippedTags: 'skipped', tags: 'run'
      }
    }
    stage('Clean Workspace after build'){
      steps{
        cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
      }
    }
 }
}
```
## Step 2: CI/CD Pipeline for a TODO Application

### Step 2.1: Configure Artifactory

1. Create an Ansible role to install Artifactory(You may install manually first on artifactory server: https://www.howtoforge.com/tutorial/ubuntu-jfrog/)

2. Run the role against the Artifactory server

### Step 2.2: Prepare Jenkins

1. Fork the `php-todo` repository (https://github.com/darey-devops/php-todo.git) into Jenkins Server

2. On you Jenkins server, install PHP, its dependencies(Feel free to do this manually at first, then update your Ansible accordingly later)
```
sudo apt install -y zip libapache2-mod-php php7.4-fpm phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip,xdebug}
```

3. Install PHP Composer manually:

```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
   
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
   
php composer-setup.php
   
mv composer.phar /usr/local/bin/composer
```

4. Configure the php.ini file, you can get the path to the file by running
```
php --ini | grep xdebug
```

5. Once you get the path to the file, open with a text editor and paste in:
```
xdebug.mode = coverage
```

6. Install nodejs and npm

```
sudo apt-get update -y
sudo apt-get install nodejs -y
sudo apt install npm -y

Install typescript using node package manager(npm). 
npm install -g typescript
```
7. Restart php7.4-fpm
```
sudo systemmctl restart php7.4-fpm
```
### Install the following plugins on Jenkins UI

>Plot plugin: to display tests reports and code coverage information

>Artifactory plugin: to easily deploy artifacts to Artifactory server
1. Go to the artifactory URL(http://artifactory-server-ip:8082) and create a local generic repository named `php-todo`(Default username and password is admin. After logging in, change the password) 
2. Configure Artifactory in Jenkins UI
3. Click Manage Jenkins, click Configure System
4. Scroll down to JFrog, click Add Artifactory Server
5. Enter the Server ID
6. Enter the URL as:
```
http://<artifactory-server-ip>:8082/artifactory
```
7. Enter the Default Deployer Credentials(the username and the changed password of artifactory)
![](https://user-images.githubusercontent.com/18899718/136073230-b5a8b3d7-56d2-4618-8b2a-30acc67d00fb.png)


## Step 2.3: Integrate Artifactory repository with Jenkins

1. On Jenkins server, install mysql-client
2. create a dummy Jenkinsfile in php-todo repo
3. In Blue Ocean, create multibranch php-todo pipeline(follow the previous steps earlier)
4. Spin up an instance for a database, install and configure mysql-server( This is where data pertaining to the artifactory repository will be stored).
5. Create a database and user on the database server
```
CREATE DATABASE homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
FLUSH PRIVILEGES;
```
6. Check if the database created on the database server can be reached from the Jenkins server. On the jenkins server, run command:
```
mysql -u <DB_user> -h <DB-private-ip-address> -p
```
7. Update the `.env.sample` file with your db connectivity details

8. Update Jenkinsfile with proper configuration
```
pipeline {
  agent any

  stages {

   stage("Initial cleanup") {
        steps {
          dir("${WORKSPACE}") {
            deleteDir()
          }
        }
      }

  stage('Checkout SCM') {
    steps {
          git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
    }
  }

  stage('Prepare Dependencies') {
    steps {
           sh 'mv .env.sample .env'
           sh 'composer install'
           sh 'php artisan migrate'
           sh 'php artisan db:seed'
           sh 'php artisan key:generate'
        }
      }
    }
  }
```
![](https://user-images.githubusercontent.com/18899718/136074120-4cbd9eea-5fcc-4a9d-a219-c7d16148941e.png)

9. Update Jenkinsfile to include unit tests
```
stage('Execute Unit Tests') {
    steps {
           sh './vendor/bin/phpunit'
    }
```

{5676EDD4-F9E3-452C-8BE3-F43492EE790A} png

## Step 2.4: Code Quality Analysis

> Most commonly used tool for php code quality analysis is phploc

1. Add the code analysis step in Jenkinsfile

        stage('Code Analysis') {
        steps {
              sh 'phploc app/ --log-csv build/logs/phploc.csv'

        }
      }

![](https://user-images.githubusercontent.com/18899718/136074559-f0d91eee-ff61-4406-b83e-71cc010659cf.png)

### Plot the data using plot Jenkins plugin.

1. This plugin provides generic plotting (or graphing) capabilities in Jenkins. It will plot one or more single values variations across builds in one or more plots. Plots for a particular job (or project) are configured in the job configuration screen, where each field has additional help information. Each plot can have one or more lines (called data series). After each build completes the plots’ data series latest values are pulled from the CSV file generated by phploc.
```
stage('Plot Code Coverage Report') {
  stage('Plot Code Coverage Report') {
      steps {

            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)                          ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'      
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes ', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
            plot csvFileName: 'plot-396c4a6b-b573-41e5-85d8-73613b2ffffb.csv', csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'
          }
        }
```

2. Click on the Plots button on the left pane. If everything was configured properly, you should see something like this:

![](https://user-images.githubusercontent.com/18899718/136075278-10c4f90f-eeb7-4bad-bb6e-a89d529ad308.png)


### Package the artifacts
```
stage ('Package Artifact') {
  steps {
          sh 'zip -qr ${WORKSPACE}/php-todo.zip ${WORKSPACE}/*'
  }
```
![](https://user-images.githubusercontent.com/18899718/136600367-69cca46a-2aeb-4978-8106-c7e09b8413d1.png)

### Publish packaged artifact into Artifactory

stage ('Deploy Artifact') {
  steps {
    script { 
      def server = Artifactory.server 'artifactory-server'
        def uploadSpec = """{
                  "files": [{
                     "pattern": "php-todo.zip",
                     "target": "php-todo"
                  }]
        }"""

               server.upload(uploadSpec) 
      }
  }
}
![](https://user-images.githubusercontent.com/18899718/136600705-e2583a85-1d95-46cc-bdc2-8f70b31a67d8.png)

For this to work, you have to create a local repository on your Artifactory with package type as Generic and Repository Key as the name of the repo (php-todo in this case) which we already did as instructed up there

### Deploy application to dev environment by launching the Ansible pipeline
```
stage ('Deploy to Dev Environment') {
  steps {
  build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
  }
}
```

The `build job` used in this step tells Jenkins to start another job. In this case it is the `ansible-config-mgt` job, and we are targeting the `main` branch. Hence, we have `ansible-config-mgt/main`. Since the Ansible project requires parameters to be passed in, we have included this by specifying the parameters section. The name of the parameter is `env` and its value is `dev`. Meaning, deploy to the `Development environment`.

## Install and Configure SonarQube on Ubuntu 20.04 With PostgreSQL as a Backend Database

Software Quality - The degree to which a software component, system or process meets specified requirements based on user needs and expectations.

Software Quality Gates - Quality gates are basically acceptance criteria which are usually presented as a set of predefined quality criteria that a software development project must meet in order to proceed from one stage of its lifecycle to the next one.

We will make some Linux Kernel configuration changes to ensure optimal performance of the tool - we will increase vm.max_map_count, file discriptor and ulimit(Check Sonarqube Documentation for details)
Step 3.1: Tune Linux Kernel

On the sonarqube server, log into to the root user and run the following commands
```
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```
Open `/etc/security/limits.conf` file and append the following.
```
sonarqube   -   nofile   65536
sonarqube   -   nproc    4096
```
To enable persistence after reboot, open `/etc/sysctl.conf` and add the following.
```
vm.max_map_count=262144
fs.file-max=65536 
```
###  Update system packages and Install Java(sonarqube is java-based) and other required packages
```
Update and upgrade packages

sudo apt-get update -y
sudo apt-get upgrade -y

Install wget and unzip packages
sudo apt-get install wget unzip -y

Install OpenJDK and Java Runtime Environment (JRE)

sudo apt-get install openjdk-11-jdk -y
sudo apt-get install openjdk-11-jre -y
```
To set default JDK or switch to OpenJDK, run the command,
```
sudo update-alternatives --config java
```

Verify JAVA is installed by running:
```
java -version
```

###  Install and Setup PostgreSQL 10 Database for SonarQube

1. In security group on AWS console, open port 5432 for postgreSQL the Sonarqube Instance

2. Add PostgreSQL to repo list
```
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
```
3. Download PostgreSQL software
```
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
```
4. Install PostgreSQL
```
sudo apt-get -y install postgresql postgresql-contrib
```
5. Start and enable PostgreSQL Database service
```
sudo systemctl start postgresql
sudo systemctl enable postgresql
```
6. Change the default password for postgres user (to any password you can easily remember)
```
sudo passwd postgres
```
7. change to postgres user
```
su - postgres
```
8. Create a new user for SonarQube
```
createuser sonar
```
9. Create Sonar DB and user. To do this, we have to change to PostgreSQL shell
```
psql
```
10. Set up encrypted password for newly created user
```
ALTER USER sonar WITH ENCRYPTED password 'sonar';
```
11. Create a DB for SonarQube
```
CREATE DATABASE sonarqube OWNER sonar;
```
12. Grant required privileges
```
grant all privileges on DATABASE sonarqube to sonar;
```
12. Exit the shell
```
\q
```
12. Switch back to initial user
```
exit
```

### Install and configure SonarQube

1. Download temporary files to /tmp folder
```
cd /tmp && sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-7.9.3.zip
```
2. Unzip archive to `/opt` directory and move to `/opt/sonarqube`
```
sudo unzip sonarqube-7.9.3.zip -d /opt
sudo mv /opt/sonarqube-7.9.3 /opt/sonarqube
```
4. Configure SonarQube Since Sonarqube cannot be run as root user, we have to create a sonar user to start the service with.
```
Create a group sonar
sudo groupadd sonar
```
5. Add a sonar user with control over `/opt/sonarqube`
```
sudo useradd -c "user to run SonarQube" -d /opt/sonarqube -g sonar sonar 
sudo chown sonar:sonar /opt/sonarqube -R
```
6. Open and edit SonarQube configuration file
```
sudo vim /opt/sonarqube/conf/sonar.properties
```
7. Find and edit the following lines
```
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
```
Add the line below
```
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

8. Edit sonar script file. Find and set RUN_AS_USER to be equals to sonar
```
sudo nano /opt/sonarqube/bin/linux-x86-64/sonar.sh

RUN_AS_USER=sonar

```

### Add sonar user to sudoers file
1. change `sonar` user password to something you can easily remember
```
sudo passwd sonar
```
2. Add sonar to sudo group
```
sudo usermod -a -G sudo sonar
```
3. To check if it has been successfully added, run
```
groups sonar
```
4. Switch to `sonar` user, and run sonar script
```
su - sonar

cd /opt/sonarqube/bin/linux-x86-64/
./sonar.sh start
```
5. Expected output:
```
Starting SonarQube...

Started SonarQube
```
6. Check SonarQube logs
```
tail /opt/sonarqube/logs/sonar.log
```
![](https://user-images.githubusercontent.com/18899718/136931954-7d0c8202-8b7c-420a-acfa-e9b8589d6b58.png)

### Configure systemd to manage SonarQube service
1. Stop the sonar service
```
cd /opt/sonarqube/bin/linux-x86-64/
./sonar.sh stop
```
2. Create systemd service file
```
sudo vim /etc/systemd/system/sonar.service
```
3. Paste in the following lines:
```
[Unit]
Description=SonarQube service
After=syslog.target network.target

[Service]
Type=forking

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

User=sonar
Group=sonar
Restart=always

LimitNOFILE=65536
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

4. Save and exit
5. Use systemd to manage the service
```
sudo systemctl start sonar
sudo systemctl enable sonar
sudo systemctl status sonar
```
![](https://user-images.githubusercontent.com/18899718/136932692-7683aad5-7ff8-4cbd-ab75-252a06214472.png)

## Access Sonarqube

1. Access the SonarQube by going to 
```
http://server_IP:9000
```
2. Use `admin` as your username and password

Note: Do not forget to open ports 9000 and 5432 on the security group.

![](https://user-images.githubusercontent.com/18899718/136933133-890d444b-b21e-46b7-b824-5c745b2d2ccc.png)

### Configure SonarQube and Jenkins for Quality Gate

1. Install SonarQube plugin in Jenkins UI

2. Navigate to Manage Jenkins > Configure System and add a SonarQube server
3. In the name field, enter sonarqube
4. For server URL, enter the IP of your SonarQube instance
5. Generate authentication tokens to use in the Jenkins UI
6. In SonarQube UI, navigate to User > My Account > Security > Generate tokens
7. Type in the name of the token and click Generate

### Configure SonarQube webhook for Jenkins
1. Navigate to Administration > Configuration > Webhooks > Create
2. Enter the name
3. Enter URL as http://jenkins-server-ip:8080/sonar-webhook/

![](https://user-images.githubusercontent.com/18899718/136933705-51f3eeaa-eede-4a47-82e2-2dd9b645079e.png)

### Setup SonarScanner for Jenkins
1. In Jenkins UI, go to Manage Jenkins > Global Tool Configuration
2. Look for SonarQube Scanner
3. Click 'Add SonarQube Scanner' and enter the scanner name as 'SonarQubeScanner'
4. Check the 'Install automatically' box
5. Select 'Install from Maven Central'

![](https://user-images.githubusercontent.com/18899718/136933996-a4d853be-c49e-48b6-8f8c-79f0dec15a04.png)

6. Save and run the pipeline to install the scanner. 
>An error will be generated but it will also create "/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/sonar-scanner.properties" directory

7. Edit sonar-scanner.properties file
```
sudo vim /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/sonar-scanner.properties
```
8. Paste in the following lines:
```
sonar.host.url=http://sonarqube-ip:9000
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/phploc.csv
sonar.php.tests.reportPath=reports/unitreport.xml
```


### Update Jenkins Pipeline to include SonarQube scanning

1. To achieve this, we have to set the environment variable for the `scannerHome`. We would use the same name used when we configured `SonarQube Scanner` from `Jenkins Global Tool Configuration`. If you remember, the name was `SonarQubeScanner`. Then, within the steps use shell to run the scanner from `bin` directory.

>To further examine the configuration of the scanner tool on the Jenkins server - navigate into the tools directory
```
cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/bin. 
```
2. List the content to see the scanner tool sonar-scanner with command `ls -latr`. That is what we are calling in the pipeline script.

### Add the following build stage for Quality Gate

```
stage('SonarQube Quality Gate') {
    environment {
          scannerHome = tool 'SonarQubeScanner'
      }
      steps {
          withSonarQubeEnv('sonarqube') {
              sh "${scannerHome}/bin/sonar-scanner"
          }

      }
  }
  
```
3. The end to end pipeline view will look something like this:
![](https://user-images.githubusercontent.com/18899718/136975660-d52126c2-2f71-4ec4-b0a4-d46e721c3279.png)
The quality gate we just included has no effect. And if you check `SonarQube` UI, you will realise that we just pushed a poor-quality code onto the development environment.

4. Navigate to your php-todo dashboard on SonarQube UI

![](https://user-images.githubusercontent.com/18899718/136975024-9ea7b722-4978-4668-b91d-22cc0a31555d.png)

### Conditionally Deploy to Higher Environments

1. Include a when condition to execute the Quality Gate stage only when the running branch is develop, hotfix, release, main
```
when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
```
2. Add a timeout step to abort to Quality Gate stage after 1 minute (or any time you like)
```
timeout(time: 1, unit: 'MINUTES') {
  waitForQualityGate abortPipeline: true
  }
```
3. Quality Gate stage snippet should look like this:
```
stage('SonarQube Quality Gate') {
    when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP"}
      environment {
          scannerHome = tool 'SonarQubeScanner'
      }
      steps {
          withSonarQubeEnv('sonarqube') {
              sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
          }
          timeout(time: 1, unit: 'MINUTES') {
              waitForQualityGate abortPipeline: true
          }
      }
  }
```

You should get the following when you run the pipeline

![](https://user-images.githubusercontent.com/18899718/136976733-3c35e7b1-a3b5-4a0c-9f68-219a79c03e1b.png)
When running a non-specified branch, SonarQube Quality Gate stage is skipped

![](https://user-images.githubusercontent.com/18899718/136976936-683736d0-cd0a-4554-b6c3-a1d1ac77327a.png)


## Configure Jenkins slave servers

1. Spin up a new EC2 Instance
2. Install all the neccessary software packages just like you did with the master(bastion) server
3. On the main Jenkins server
4. Navigate to Manage Jenkins > Manage Nodes
5. Click New Node
6. Enter name of the node and click the 'Permanent Agent' button and click the OK button
7. Fill in the remote root directory as /home/ubuntu
8. Set 'Host' value as the Public-IP of the slave node
9. For Launch Method, select Launch Agents via SSH
10. Add SSH with username and private key credentials with username as ubuntu and private key as the private key of the master node
11. For Host Key Verification Strategy, select Manually trusted key validation strategy
12. Click Save 
![](https://user-images.githubusercontent.com/18899718/136978657-abd22c46-85ec-4d0a-b112-f6c3cc201d0d.png)

![](https://user-images.githubusercontent.com/18899718/136977446-248d0319-d5dd-4caf-8ca6-88adcd529bd1.png)



