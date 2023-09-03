## Continued from Project 11

# Step 1 Jenkins job enhancement
### Every new change in the codes creates a build which in turn creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins serves with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will require Copy Artifact plugin.


- Go to your Jenkins-Ansible server and create a new directory called ansible-artifact – we will store there all artifacts after each build.

`sudo mkdir /home/ubuntu/ansible-artifact`
- Change permissions to this directory, so Jenkins could save files there

`chmod -R 0777 /home/ubuntu/ansible-artifact`

![](./project12%20images/ansible-artifact%20created.png)

- Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

- Create a new Freestyle project and name it save_artifacts.

- This project will be triggered by completion of your existing ansible project. Configure it accordingly

- Configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

![](./project12%20images/configure%20copy%20artifact.png)

### The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-artifact directory. To achieve this, create a Build step and choose Copy artifacts from other project, specify ansible as a source project and /home/ubuntu/ansible-artifact as a target directory.

- Test your set up by making some change in README.MD file inside your complete-ansible-automation repository (right inside master branch).

- If both Jenkins jobs have completed one after another – you shall see your files inside /home/ubuntu/ansible-artifact directory and it will be updated with every commit to your master branch.

![](./project12%20images/saved%20artifact%20worked1.png)

![](./project12%20images/saved%20artifact%20worked2.png)

- Now your Jenkins pipeline is more neat and clean.

# Step 2: Refactor Your Code

### In Project 11 you wrote all tasks in a single playbook common.yml, you will refactor your code by breaking tasks up into different files. This is an excellent way to organize complex sets of tasks and reuse them.

### First create a new branch called `Refactor and checkout into the branch`

- Within playbooks folder, create a new file and name it site.yml – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, site.yml will become a parent to all other playbooks that will be developed. Including common.yml that you created previously.

- Create a new folder in root of the repository and name it static-assignments. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work

- Move common.yml file into the newly created static-assignments folder.

- Inside site.yml file, import common.yml playbook.

![refactored code](./project12%20images/refactored%20code.png)

### Run ansible-playbook command against the dev environment

### Since wireshark is already installed, create a new playbook called `common_del.yml` and uninstall wireshark from your servers

### checkout to your main, pull the changes down and run the playbook

`ansible-playbook -i inventory/dev.yml playbooks/site.yaml`

### Make sure that wireshark is deleted on all the servers by running `wireshark --version`

![](./project12%20images/wireshark%20installed.png)

![](./project12%20images/delete_wireshark.png)

![](./project12%20images/wireshark%20deleted.png)

## Your current architecture will be:

![](./project12%20images/architecture%20after%20refactoring.png)


# STEP 3: Configure uatwebservers

### your dev environment is neat and clean. Now work with your uat environment.

### Instead of writing numerous tasks in same playbook, you can use roles. Roles break down these tasks and makes them reuseable

### Launch 2 new instances and name them Web1-UAT and Web2-UAT. add them to your uat inventory file

### To create a role, you must create a directory called roles/, then proceed to use the ansible-galaxy utility to create the role

`mkdir roles`

`cd roles`

`ansible-galaxy init webserver`

### Provide your role path in your ansuble.cfg file so Ansible could know where to find configured roles

![](./project12%20images/roles_path.png)

### Now it's time to add logic to your webserver role, go into tasks directory, and within the main.yml file, start writing configuration tasks to do the following:

- Install and configure Apache (httpd service)
Clone Tooling website from GitHub https://github.com/NyerhovwoOnitcha/tooling.git

- Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
- Make sure httpd service is started

![](./project12%20images/tasks%20.png)

### Within the static-assignments folder, create a new assignment for uat-webservers uat-webservers.yml. This is where you will reference the role.

![](./project12%20images/uat_webserver_yml.png)

![](./project12%20images/reference%20webserver%20role.png)

### Commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs and they ran successfully.
### Now run the playbook against your uat inventory and see what happens:

`ansible-playbook -i inventory/uat.yml playbooks/site.yml`

![](./project12%20images/webserver%20role%20successful.png)

### You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

#### http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php

![](./project12%20images/webserver%20role%20successful2.png)


