# Session 1

- [What is Ansible?](#what-is-ansible)
- [Why should we use Ansible?](#why-should-we-use-Ansible)
- [When should we use Ansible?](#when-should-we-use-ansible)
- [Ansible commands](#ansible-commands)
- [Ansible architecture](#ansible-architechture)
- Ansible components:
  - [Ansible glossary](#glossary)
  - [Playbooks, play, tasks](#playbooks)
  - [Modules](#modules)
  - [Roles](#roles)
  - [Jinja2](#jinja2)
  - [Variables](#variables)
  - [Templating](#templating)
  - [Conditionals](#conditionals)
  - [Loops](#loops)
  - [Inventory file](#inventory-file)
  - [Configuration file](#configuration-file)
  - [Vault](#vault)
- [Ansible galaxy](#ansible-galaxy)
- [Best practices](#best-practices)

## What is Ansible

Ansible is a modern IT automation tool.  
It helps to make our lives easier by managing our servers for us. We just need figure out what needs to be done in our servers, how it should be done, and then use Ansible to write the flow of operations that should run in our servers, and Ansible will be executing the tasks for us. It requires to spend some initial time do define our configuration and the rest will be only maintaining it by any member in the team.  
Don't forget that managing 1 server is easy, managing 5 servers is do’able, but managing hundreds or more is a painful task without automation.

## Why should we use Ansible

Among others features, Ansible is:
- **Human readable**: It uses YAML because it is easier for humans to read and write than other common data formats (XML, JSON).
- **No special coding skills**: Again, Ansible uses YAML, which is very intuitive and most likely everyone in our time will be able to read and update very easily whitout consuming a lot of time.
- **Tasks executed in order**: With this you can be sure that the flow is going to stay exactly as you define and there will be no surprises.
- **Powerful**: It can orchestrate infrastructure with very detail control over the machines.
- **Flexible**: It can adjust easily to your needs.
- **Positive team impact**: Ansible eliminate repetitive tasks, which translates into saving time and being more productive, having fewer mistakes and improve collaboration in the team.

## When should we use Ansible

As presented on the Ansible [website](https://www.ansible.com/use-cases-overview), it is recommended in the following cases:
- **Provisioning**: Wherever is the condition our the machines to set up (bare-metal servers, virtual machines, cloud instances), Ansible sets up the system with all required services and technologies.
- **Configuration management**: Ansible is consistent, secure and highly reliable, with an extremely low learning curve for administrators, developers and IT managers.
- **Application deployment**: Playbooks are repeatable, reliable, simple to write and maintain. Besides that, Ansible is agentless, so it can be introduced into the system without any bootstrap system or opening up additional ports.
- **Continuous delivery**: Automation and simplicity are offered by Ansible. It provides rolling updates with zero downtime thanks to its push-based architecture, having very fine grained control over operations to orchestrate configuration of servers in batches, all while working with load balancers, monitoring systems, and cloud or web services.
- **Security and Compliance**: Using automation tooling allows you to apply security in a simple and consistent manner
- **Orchestration**: Deployments on clusters, clouds. Orchestration is about bringing together disparate things into a coherent whole.

## Ansible commands

### ansible

_Lets first run our first command which pings our localhost machine._

```sh
ansible localhost --module-name ping
```
Ansible outputs the following message:
```
localhost | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
```
In case you don't see this, but instead:
```
[WARNING]: Could not match supplied host pattern, ignoring: localhost

[WARNING]: No hosts matched, nothing to do
```
Please modify your `/etc/ansible/hosts` file with the following line:
```sh
localhost ansible_connection=local
```
```
localhost | FAILED! => {
  "changed": false,
  "failed": true,
  "msg": "file (/tmp/test-file) is absent, cannot continue",
  "path": "/tmp/test-file",
  "state": "absent"
}
```

_If now we want to try to access one file, we need to use the module `file`, which receives arguments, we will execute:_
```sh
ansible localhost -m file -a "path=/tmp/test-file state=file"
```
When the file does not exist we get:
```
localhost | FAILED! => {
    "changed": false,
    "failed": true,
    "msg": "file (/tmp/test-file) is absent, cannot continue",
    "path": "/tmp/test-file",
    "state": "absent"
}
```
If the file does exist we get:
```
localhost | SUCCESS => {
    "changed": false,
    "failed": false,
    "gid": 0,
    "group": "wheel",
    "mode": "0644",
    "owner": "mariadominguez",
    "path": "/tmp/test-file",
    "size": 2048,
    "state": "file",
    "uid": 0
}
```

### ansible-playbook

This is the command we will use all over the session 1 of the workshop, so more examples will appear later.

### ansible-config

_Show the list of configuration options and their specification_
```sh
ansible-config list
```

_Show the configuration file_
```sh
ansible-config view
```

:exclamation: **NOTE**: If installing Ansible from a package manager, the latest `ansible.cfg` should be present in `/etc/ansible`, possibly as a `.rpmnew` file (or other) as appropriate in the case of updates. If you have installed from `pip` or from source, however, you may want to create this file in order to override default settings in Ansible. In that case, please download the latest version of the Ansible configuration file from https://raw.githubusercontent.com/ansible/ansible/devel/examples/ansible.cfg and save it into your home directory as `~/.ansible.cfg`.

_Show the actual values of the Ansible configuration_
```sh
ansible-config dump
```

### ansible-console

_Run `ansible-console` and run any module._
```
$ ansible-console
Welcome to the ansible console.
Type help or ? to list commands.

mariadominguez@all (1)[f:5]$ ping
localhost | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
mariadominguez@all (1)[f:5]$ file path=/tmp/test-file state=file
localhost | SUCCESS => {
    "changed": false,
    "failed": false,
    "gid": 0,
    "group": "wheel",
    "mode": "0644",
    "owner": "mariadominguez",
    "path": "/tmp/test-file",
    "size": 2048,
    "state": "file",
    "uid": 501
}
mariadominguez@all (1)[f:5]$ exit
```

### ansible-doc

```sh
ansible-doc ping
ansible-doc file
```

Alternatively, if you have connection to the Internet, you can access http://docs.ansible.com/ansible/latest/modules_by_category.html and search for the specific module you want to use.

### ansible-galaxy

More info about Ansible Galaxy later on.

### ansible-vault

_Create an encrypted file. When you run below command, Ansible will ask you to enter the encryption password for the encrypted file you are creating. Once entered the password, you can manually add the content to the file._
```sh
ansible-vault create password
```
Observe the content has been encrypted (`$ANSIBLE_VAULT;1.1;AES256`):
```sh
less test.yml
```
_Decrypt the password file we just created. Ansible asks for the encryption password. Once provided, the file is decrypted._
```sh
ansible-vault decrypt --ask-vault-pass password
```
Observe the content has been decrypted:
```sh
less password
```
_An existing file can be encrypted as well. Ansible asks for the encryption password. Once provided, the file is encrypted._
```sh
ansible-vault encrypt test.yml
```
_An encrypted file can also be edited and remain encrypted. Ansible asks for the encryption password. Once provided, the file will be open for edition. Once saved, the content is automatically encrypted again._
```sh
ansible-vault edit test.yml
```

## Ansible architecture

![ansible-architecture](https://github.com/lunatech-labs/ansible-workshop/blob/master/docs/ansible-architechture.png)

The **Host Inventory** file determines the target machines where these plays will be executed. The remote servers should have Python installed. The remote machines can live in the cloud, bare-metal, vm's, etc. When accessing machines on the cloud, a dynamic inventory file is used, otherwise a static inventory file must be provided by the users.

The **Ansible configuration** file can be customised to reflect the settings in your environment.

**Users** can execute commands on remote machines or run a sequenced instruction set via **Playbooks**.

The **Playbooks** consist of one or more tasks that are expressed either with **Core Modules** that come with Ansible or **Custom Modules** that users can write for specific situations.

**Plugins** allow to execute Ansible tasks as a job build step. For example: cache plugins are used to keep a cache of 'facts' to avoid costly fact-gathering operations.

## Glossary

:information_source: For the scope of this workshop, we are focusing on the following terms. Visit http://docs.ansible.com/ansible/latest/glossary.html for an official extended version of the glossary.

A **playbook** is composed of 1 or more **plays** in a list.

A **play** maps a group of hosts to some well defined **roles**.

A **role** is composed by a set of **tasks**.

A **task** is a call to an Ansible **module**.

A **module** defines the actual execution to be run in the host.

## Playbooks

A Playbook defines the description of actions that you want to apply to your system. For example:

```yaml
---
- hosts: localhost
  tasks:
    - name: Create a directory
      file: path="./new-directory" state=directory
```
We run a playbook with `ansible-playbook`. Adding one level of verbosity (`-v`) is recommended, but more can be added for debugging purposes.

```sh
ansible-playbook playbook.yml -v
```

## Modules

A module in Ansible is the unit of work shipped out to the remote machines. They can be executed by `ansible` command or by `ansible-playbook` as part of the plays.

The execution of a module returns a JSON. Examples:

```
ansible localhost -m ping
localhost | SUCCESS => {
    "changed": false,
    "failed": false,
    "ping": "pong"
}
```

```
ansible-playbook create-directory.yml -v
[...]
TASK [Create a directory] *******************************************************************************************************************************************************************************
changed: [localhost] => {"changed": true, "failed": false, "gid": 20, "group": "staff", "mode": "0755", "owner": "mariadominguez", "path": "new-directory", "size": 68, "state": "directory", "uid": 501}
[...]
```

## Roles

Roles are a structured of grouping tasks that implement a specific functionality on a host. Ansible requires a specific structure in the way a role is defined as per the below:

```
.
├── defaults
│   └── main.yml
├── files
│   └── file-name.txt
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   └── main.yml
├── templates
│   └── template-name.j2
└── vars
    └── main.yml
```

Considerations:

- In general, at least the `tasks/main.yml` should exist for the role to make sense. It will contain the list of task to be executed when the role is called.
- `files` contains files that need to be used by the tasks to perform their action on the host.
- `templates` contains files with variables whose value will be substituted during runtime and will be used by the tasks to perform their action on the host.
- `defaults` provides default variables for the role.
- `variables` provides other variables for the role. If the same variable appears in both `defaults` and `variables`, the one in `variables` has preference.
- `meta` contains metadata for the role. Any role dependencies defined in this folder will be run before any other task in the role.
- `handlers` contains tasks that will be executed as the last step in the playbook. They are referenced by a globally unique name.
- Apart from the `main.yml`, each directory can contain more `.yml` files (`files` and `templates` will contain any kind of files)

By default, each role can be run only once, even when it appears more than once in a playbook. For example:
```yaml
---
- hosts: all
  roles:
    - common
    - application
    - application
```
will produce a single execution of the role `application`. If we want to run the role more than once, there are two options:
1. Add `allow_duplicates: true` to `meta/main.yml`.
2. Pass different parameters to the role. For example:
  ```yaml
  ---
  - hosts: all
    roles:
      - common
      - { role: application, dir: '/opt/a' }
      - { role: application, dir: '/opt/b' }
  ```

## Jinja2

Jinja2 is a modern and designer-friendly templating language for Python. More info can be found in http://jinja.pocoo.org/docs/2.10. Ansible uses Jinja2 when applying filters to variables and templates.

### Tests

Tests in Jinja2 are a way of evaluating template expressions and returning True or False. Built-in tests can be found in http://jinja.pocoo.org/docs/2.10/templates/#builtin-tests.

Tests always execute on the Ansible controller, not on the target of a task, as they test local data.

## Variables

Variables in Ansible can be defined in many different places, and if one variable appears in more than one place, there is a precedence rule that applies. Extra variables passed by command line always win in precedence.

### Filters

Filters in Jinja2 are a way of transforming template expressions from one kind of data into another. They are applied using the pipe (`|`) when using a variable. For example:

```yaml
---
- hosts: all
  vars:
    input: "{{ raw_data | to_json }}"
```

## Templating

Ansible can use templates as part of the role. All the templating will be executed on the Ansible controller before the task is sent to the target machine to minimize the requirements on the remote host.

On http://jinja.pocoo.org/docs/2.10/templates/ we can find the documentation to write templates that will be processed in Ansible. For example:

```yaml
# ./roles/common/templates/app.conf.j2

{% if app_args is defined %}args={{ app_args | replace(",", " ") }}{% endif %}

base_url=http://example.com
user=maria
role=presenter
```

```yaml
# ./roles/common/vars/main.yml

---
user: "Admin"
app_args: "arg1=value1,arg2=value2"
```

```yaml
# ./roles/common/tasks/main.yml
---
- name: Copy configuration to the host
  template:
    src: "../templates/app.conf.j2"
    dest: "/tmp/app.conf"
    mode: 0644
```

```yaml
# ./common-playbook.yml
---
- hosts: localhost
  roles:
    - common
```

creates the file `/tmp/app.conf` with the content below:

```ini
args=arg1=value1 arg2=value2
base_url=http://example.com
user=maria
role=presenter
```

## Conditionals

Whether to run a task or not, can be decided by using `when`. For example:

```yaml
---
- hosts: localhost
  tasks:
    - name: List content of home directory
      shell: ls -l ~
      when: ansible_distribution == "MacOSX"
```

## Loops

If a specific task need to be performed equally in more than one case, we can use `with_items` to loop over all the elements. For example:

```yaml
---
- hosts: localhost
  tasks:
    - name: Print all elements in the list
      command: echo {{ item }}
      with_items: [0, 2, 4, 6, 8, 10]
```

There are many variants of `with_items`, based on the type of data that we are managing. Among the most common:
- `with_dict`
- `with_file`
- `with_together`
- `with_lines`
- `with_first_found`

Take a look at http://docs.ansible.com/ansible/latest/playbooks_loops.html to find all the possible looping options.

## Inventory file

In order for Ansible to be able to connect to remote servers, their configuration should be defined in a file, that is the Inventory file. Depending on whether the hosts are in the cloud or not, there are two types of inventory files: static and dynamic.

If your instances live in the cloud, for example, in Amazon EC2, it is recommended that you configure your inventory file as dynamic. This process is not covered in the workshop, but on the Ansible website you will find a [section](http://docs.ansible.com/ansible/latest/intro_dynamic_inventory.html) with all the details to configure you Dynamic Inventory file.

Otherwise, you will use a regular Inventory file, usually in INI format, where you configure which are the ones that Ansible is going to connect to. The layout of the file will be something similar to:

```ini
[dev]
master ansible_host=master.dev.com ansible_port=22 ansible_user='admin'
slave1 ansible_host=slave1.dev.com ansible_port=22 ansible_user='admin'
slave2 ansible_host=slave2.dev.com ansible_port=22 ansible_user='admin'
```

You can give any group name you want (as `dev` in the example), but Ansible provides two default groups available for you to use: `all`, which includes all the hosts defined in the inventory file, and `ungrouped`, which includes all those hosts that do not belong to any group (apart from group `all`).

Which parameters are configurable for a host is available on http://docs.ansible.com/ansible/latest/intro_inventory.html#list-of-behavioral-inventory-parameters.

Although is also possible to assign variables to groups of hosts in the inventory file (using `[my_group_name:vars]`), Ansible recommends not to do it, but instead define them in a separate file under `group_vars` directory, with the name of the group. The same applies for host, with `host_vars` directory.

In this case, the layout of a project which contains 3 groups of hosts (`dev`, `test` and `prod`) will look similar to:

```
.
├── group_vars
│   ├── all
│   ├── dev
│   ├── test
│   └── prod
├── roles
│   └── common
│   └── ...
├── dev
├── test
└── prod
```

## Configuration file

There are plenty of configuration options to be customised. Defaults should work fine in most of the cases, but once you get familiar with Ansible, you can read through all of them and consider if some changes are worth in your particular case.

Don't forget all the places where Ansible can check for configuration, and their precedence!

## Vault

Take a look at the section [ansible-vault](#ansible-vault) where we have already seen some examples for the Ansible vault feature.

If you have other needs for encryption, please refer http://docs.ansible.com/ansible/latest/vault.html for more information, because this workshop is not focused on this feature.

## Ansible Galaxy

Since Ansible is widely used, there are many already implemented roles that you can reuse in you system. Take a look at https://galaxy.ansible.com/ and search for the ones you may need. If you think you can contribute with a role, join the community and share!

## Best practices

The fundamental one is: Keep it simple. Because Ansible is very flexible and you can achieve the same goal in many different ways, you should choose always the simpler one. Think about others having to maintain your code (or you maintaing others' code).

Other recommendations:
- Use the recommended directory layout (http://docs.ansible.com/ansible/latest/playbooks_best_practices.html#directory-layout). Having a common understanding of the structure of the project will allow anybody in the team to easily find and update any files.
- Keep your inventory well organised. Make meaningful groups of hosts based on the implementation that should be done on each of the groups.
- Define groups and hosts variables under `group_vars` and `host_vars`. Use files `all` to include all the hosts, or the group/host name to refer a specific group/host.
- Having groups of hosts in the inventory, and their specific variables in the correspondent directories, would be completed with grouping your tasks in roles and playbooks based on your hosts groups. This comes with the advantage of limiting runs to a specific set of hosts (for example: `ansible-playbook site.yml --limit webservers`).
- If your module contains the parameter `state`, try to use it as much as possible, because it will help to make clear what should occur at a specific task after some time not going through the tasks in the playbook.
- The same applies to `creates` and `removes`. In this case you can avoid performing tasks when the result of them is already present in the target machines.
- Always give a name to your tasks. It will help to understand what the module is trying to achieve.
- Use at least the first level of verbosity (`-v`). This will allow you to easily go through the output of any task and review the work that has been executed in the remote machines.
