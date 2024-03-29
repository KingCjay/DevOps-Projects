### Ansible Dynamic Assignments (Include) and Community Roles

Unleash flexiblilty of Ansible with dynamic assignments (include) and community roles in this project, expanding your automation capabilities and leveraging pre-built solutions.

# Ansible dynamic assignments (include) and community roles

**IMPORTANT NOTICE**: It's encouraged to visit `Ansible Documentation` for the latest updates on modules and their usage because Ansible is actively developing software project.

Having been equipped with some knowledge and skills from the last two projects on Ansible, to perform configurations using `playbooks`, `roles` and `imports`. Now we will continue configuring our UAT servers learning and practicing new Ansible concepts and modules.

In this project we will introduce `dynamic assignments` by using `include` module.

So whats the difference between **static** and **dynamic** assignments?

Well, from the last project, you can tell  that static assignments use `import` Ansible module. The module that enables dynamic assignments is `include`.

Hence,

```php
import = Static
include = Dynamic
```
When the **import** module is used, all statements are pre-processed at the time playbooks are parsed. Meaning,when you execute `site.yml` playbook, Ansible will process all the playbooks referenced during the time it is parsing statements. This also means that, during actual execution, if any statement changes, such statements will not be considered. Hence,it is static.

On the other hand,when **include** module is used,all statements are processed only during execution of the playbook. Meaning,after the statements are **parsed**,any changes to the statements encountered during execution will be used.

Take note that in most cases it is recommended to use static assignments for playbooks,because it is more reliable. With dynamic ones,it is hard to debug playbook problems due to its dynamic nature. However,you can use dynamic assignments for enviroment specific variables as we will be intoducing in this project.

## Introducing dynamic assignment into our structure

Introducing dynamic Assignment Into Our Structure

In your `https://github,com/<your-name>/ansible-config-mgt` GitHub repository start a new branch called `dynamic-assignments`.

Create a new folder,name it `dynamic-assignments`. Then inside this folder,create a new file and name it `env-vars.yml`. We will instruct `site.yml` to `include`. this playbook later. For now,let us keep building up the structure.

Your GitHub shall have the following structure by now.

**Note**: Depending on what method you used in the previous project you may have or not have `roles` folder in your GitHub repository - if you used `ansible-galaxy`,then `roles` directory was only created on your `Jenkins-Ansible` server locally. It is recommended to have all the codes managed and tracked in GitHub,so you might want to recreate the structure manually in this case - it is up to you.

Copy Below Code
```
├── dynamic-assignments
│   └── env-vars.yml
├── inventory
│   └── dev
    └── stage
    └── uat
    └── prod
└── playbooks
    └── site.yml
└── roles (optional folder)
    └──...(optional subfolders & files)
└── static-assignments
    └── common.yml
```

Since we will be using the same Ansible to configure multiple enviroments, and each of these enviroments will have certain unique attributes,such as **servername**,**ip-address** etc.,we will need a way to set values to variables per specific enviroment.

For this reason,we will now create a folder to keep each enviroment's variables file. Therefore,create a new folder `env-vars`, then for each enviroment,create new YAML files which we will use to set variables.

Your layout should now look like this.

Copy Below Code
```php
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

![Alt text](images/create-dynamic-folder.png)

Now paste the instruction below in the `env-vars.yml` file.

Copy Below Code
```php
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

![Alt text](images/env-vars.png)

Notice 3 things here:
1. We used `include_vars` syntax instead of `include`, this is because Ansible developers decided to seperate different features of the module. From Ansible version **2.8**, the `include` module is deprecated and variants of `include_*` must be used. These are:

- include_role
- include_tasks
- include_vars

In the same version, variants of **import** were also introduced,such as:

- import_role
- import_tasks

2. We made use of special variables `{{ playbook_dir }}` and `{{ inventory_file }}`. `{{ playbook_dir }}` will help Ansible determine the location of the running playbook, and from there navigate to other path on the filesystem. `{{ inventory_file }}`  on the other hand will dynamically resolve to the name of the inventory file being used, then append `.yml` so that it picks up the required file within the `env-vars` folder.
3. We are including the variables using a loop. `with_first_found` implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an enviroment specific env files does exist.

#### Update site.yml with dynamic assignments 

Update `site.yml` file to make use of dynamic assignment.(***At this point,we cannot test it yet. We are just setting the stage for what is yet to come.***)

**site.yml** should now look like this.

Copy Below Code
```php
---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml

```

![Alt text](images/image-2.png)

## Update site.yml with dynamic assignments

#### Community Roles

Now we create a role for MySQL database- it should install the MySQL package,create a database and configure users. But why should we re-invent the wheel? There are tons of roles that have already been developed by other open source engineers out there. The roles are actually production ready, and dynamic to accomodate most of linux flavours. With Ansible Galaxy again, we can simply download a ready to use ansible role, and keep going.

#### Download MySQL Ansible Role

We will be using a MySQL role developed by `geerlingguy`.

**Hint**: To preserve your GitHub in actual state after you install a new role - make a commit and push to master your 'ansible-config-mgt' directory. Of course must have `git` installed and configured on `Jenkins-Ansible` server and,for more convinient work with codes,you can configure Visual Studio Code to work with this directory. In this case,you will no longer need webhook and Jenkins job to update your codes on `Jenkins-Ansible` server,so you can disable it - we will be using Jenkins later for a better purpose.

On `Jenkins-Ansible` server make sure that `git` is installed with `git --version`,then go to 'ansible-config-mgt'directory and run

Copy Below Code
```php
git init
git pull https://github.com/<your-name>/ansible-config-mgt.git
git remote add origin https://github.com/<your-name>/ansible-config-mgt.git
git branch roles-feature
git switch roles-feature
```

![Alt text](images/git-switch-roles.png)

Inside `roles` directory create your new MySQL role with `ansible-galaxy installgeerlingguy.mysql` and rename the folder to `mysql`

Copy Below Code
`mv geerlingguy.mysql/ mysql`

Read `README.md` file,and edit roles configurattion to use correct credentials for MySQL required for the `tooling` website.

Now it is time to upload the changes into your GitHub:

Copy Below Code
`git add .`
`git commit -m "Commit new role files into GitHub"`
`git push --set-upstream origin roles-feature`

![Alt text](images/git-init-add.png)

Now, if you are satisfied with your codes, you create a Pull Request and merge it to `main` branch on GitHub.

#### Load Balancer roles

We want to be able to choose which Load Balancer to use,`Nginx` or `Apache`,so we need to have two roles respectively:

1. Nginx
2. Apache

With your experience on Ansible so far you can:

- Decide if you want to develop your own roles, or find available ones from the community
- Update both `static-assignment` and `site.yml` files to refer the roles.

***Important Hints***:

- Since you cannot use both **Nginx** and **Apache** load balancer,you need to add a condition to enable either one-this is where you can make use of variables.

- Declare a variable in `defaults/main.yml`file inside the Nginx and Apache roles. Name each variables `enable_nginx_lb`and `enable_apache_lb`respectively.

- Set both values to false like this `enable_nginx_lb: false` and `enable_apache_lb: false`.

- Declare another variable in both roles `load_balancer_is_required` and set its value to `false` as well.
  
- Update both assignment and site.yml files respectively

`loadbalancers.yml` file

Copy Below Code
```php
- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
```

`site.yml` file

Copy Below Code
```php
     - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 
```

![Alt text](images/static-db-yml.png)

Now you can make use of `env-vars/uat.yml` file to define which loadbalancer to use in UAT enviroment by setting respective enviromental variable to `true`.

You will activate loadbalancer,and enable `nginx` by setting these in the respective enviroment's env-vars file.

Copy Belw Code
```php
enable_nginx_lb: true
load_balancer_is_required: true
```

![Alt text](images/ansible-playbook1.png)

![Alt text](images/ansible-playbook2.png)

![Alt text](images/ansible-playbook3.png)

![Alt text](images/ansible-site-yml.png)

The same must work with `apache`LB,so you can switch it by setting respective enviromental variable to `true`and other to `false`.

To test this, you can update inventory for each environment and run Ansible against each environment.

