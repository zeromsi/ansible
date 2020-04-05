# ansible

Ansible uses ssh client to perform jobs to the nodes. Ansible is agent less. That is why ansible needs to be installed on the controll machine only. But for some of module python is needed to be install on the remote machines.

Install ```Ansible``` on ```ubuntu```

```
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

To check if ansible installed successfully,

```
ansible localhost -m ping
```


Output will be something loke following,

```
localhost | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```


## Configuring ansible

Our ansible controll machine must have the privileges to log in to the nodes and have the sudo privileges. We don't want to remember the IP address of the nodes as this is very difficult. So what we can do, we can provide a domain name to the nodes. To do this, we need to edit hosts file of our controll machine.

``` 
vi /ect/hosts

```
```hosts``` file contains contents like following,

![hosts-default](images/1.png)

Add server as following,

![hosts-default](images/2.png)

Now check if you can poing ci

``` ping ci ```


Now generate a ssh key pair in controll machine. a public and a private key will be generated. Public key wil be distributed to the nodes.

```Note:``` Do not distribute your private key.

Let's generate a ssh key pair.

``` ssh-keygen```
Press ```ENTER``` several times. ssh key parin will be generated.
To check if ssh key has been generated,

Go to your users directory,

``` ls .ssh```
You'll see two files named, ```id_rsa``` and ```id_rsa.pub```

```Note: ssh-keygen``` will generate keys for the current user.


Now try to copy the publoic key to the working nodes like following.

``` 
ssh-copy-id -i .ssh/id_rsa.pub klover-dev@ci
```

```Note:``` an error saying ``` ssh: connect to the host ubuntu port 22: Connection refused.

This means ssh client is not installed on that working node (ubuntu machine). To install ssh client, login to the working node and run following command,

``` sudo apt-get install openssh-server ```

Now try to copy public key again from ansible controll machine.


If everything goes alright, you may ssh into that node from controll machine by following command without password, 

```
ssh klover-dev@ci
```

User must be a sudo user in each nodes, to make user sudo user,

ssh to that node from controll node, run,

``` sudo visudo``` for ```ubuntu``` and ```su -``` then ```visudo``` for ```centos```
A file will be opened. This is the file where you can set permissions to the users.

Add a line like following to this file, then save.

``` 
klover-dev ALL=(ALL) NOPASSWD: ALL
```

## Ansible Inventory

```Inventroy ``` file contains the information of remote machines in ``` individual``` host format, ```group``` and ```range``` host format. 

```Ansible``` inventory file is placed at ```/etc/ansible``` directory. The name is ```hosts```.

You can add indiviual remote machine as following,

```
ci ansible_user=klover@dev
machine2 ansible_user=machine2_user
```
You can add remote machines as group

```
[Infra]
ci ansible_user=klover@dev

[worker]
machine2 ansible_user=machine2_user
```


You can use range to define nodes as well

```
[Infra]
ci[01:05]
```
The above range says, there's 5 nodes under Infra; ci01,ci02,ci03,ci4,ci05.

There's some other [keys](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html) like ```ansible_user```, we can use.



Test if your setup is okay,

``` 
ansible -m ping all
```


Let's explore ```ansible playbook```,

``` Playbook``` is all about play.

```Play struct (basic)```

```xml

play{

    hosts[],
    become,

    tasks[
        {
            name,
            ignore_errors,
            notify,
        }
    ],
    handlers[]

}

```
```Play Python class; attributes```
```xml
 class Play(Base, Taggable, CollectionSearch):

    """
    A play is a language feature that represents a list of roles and/or
    task/handler blocks to execute on a given set of hosts.
    Usage:
       Play.load(datastructure) -> Play
       Play.something(...)
    """

    # =================================================================================
    _hosts = FieldAttribute(isa='list', required=True, listof=string_types, always_post_validate=True, priority=-1)

    # Facts
    _gather_facts = FieldAttribute(isa='bool', default=None, always_post_validate=True)
    _gather_subset = FieldAttribute(isa='list', default=(lambda: C.DEFAULT_GATHER_SUBSET), listof=string_types, always_post_validate=True)
    _gather_timeout = FieldAttribute(isa='int', default=C.DEFAULT_GATHER_TIMEOUT, always_post_validate=True)
    _fact_path = FieldAttribute(isa='string', default=C.DEFAULT_FACT_PATH)

    # Variable Attributes
    _vars_files = FieldAttribute(isa='list', default=list, priority=99)
    _vars_prompt = FieldAttribute(isa='list', default=list, always_post_validate=False)

    # Role Attributes
    _roles = FieldAttribute(isa='list', default=list, priority=90)

    # Block (Task) Lists Attributes
    _handlers = FieldAttribute(isa='list', default=list)
    _pre_tasks = FieldAttribute(isa='list', default=list)
    _post_tasks = FieldAttribute(isa='list', default=list)
    _tasks = FieldAttribute(isa='list', default=list)

    # Flag/Setting Attributes
    _force_handlers = FieldAttribute(isa='bool', default=context.cliargs_deferred_get('force_handlers'), always_post_validate=True)
    _max_fail_percentage = FieldAttribute(isa='percent', always_post_validate=True)
    _serial = FieldAttribute(isa='list', default=list, always_post_validate=True)
    _strategy = FieldAttribute(isa='string', default=C.DEFAULT_STRATEGY, always_post_validate=True)
    _order = FieldAttribute(isa='string', always_post_validate=True)
```


```Base Python class; attributes```

```xml

class Base(FieldAttributeBase):

    _name = FieldAttribute(isa='string', default='', always_post_validate=True, inherit=False)

    # connection/transport
    _connection = FieldAttribute(isa='string', default=context.cliargs_deferred_get('connection'))
    _port = FieldAttribute(isa='int')
    _remote_user = FieldAttribute(isa='string', default=context.cliargs_deferred_get('remote_user'))

    # variables
    _vars = FieldAttribute(isa='dict', priority=100, inherit=False, static=True)

    # module default params
    _module_defaults = FieldAttribute(isa='list', extend=True, prepend=True)

    # flags and misc. settings
    _environment = FieldAttribute(isa='list', extend=True, prepend=True)
    _no_log = FieldAttribute(isa='bool')
    _run_once = FieldAttribute(isa='bool')
    _ignore_errors = FieldAttribute(isa='bool')
    _ignore_unreachable = FieldAttribute(isa='bool')
    _check_mode = FieldAttribute(isa='bool', default=context.cliargs_deferred_get('check'))
    _diff = FieldAttribute(isa='bool', default=context.cliargs_deferred_get('diff'))
    _any_errors_fatal = FieldAttribute(isa='bool', default=C.ANY_ERRORS_FATAL)
    _throttle = FieldAttribute(isa='int', default=0)

    # explicitly invoke a debugger on tasks
    _debugger = FieldAttribute(isa='string')

    # Privilege escalation
    _become = FieldAttribute(isa='bool', default=context.cliargs_deferred_get('become'))
    _become_method = FieldAttribute(isa='string', default=context.cliargs_deferred_get('become_method'))
    _become_user = FieldAttribute(isa='string', default=context.cliargs_deferred_get('become_user'))
    _become_flags = FieldAttribute(isa='string', default=context.cliargs_deferred_get('become_flags'))
    _become_exe = FieldAttribute(isa='string', default=context.cliargs_deferred_get('become_exe'))

    # used to hold sudo/su stuff
    DEPRECATED_ATTRIBUTES = []

```

```Task Python class; attributes```

```xml

class Task(Base, Conditional, Taggable, CollectionSearch):

    """
    A task is a language feature that represents a call to a module, with given arguments and other parameters.
    A handler is a subclass of a task.
    Usage:
       Task.load(datastructure) -> Task
       Task.something(...)
    """

    # =================================================================================
    # ATTRIBUTES
    # load_<attribute_name> and
    # validate_<attribute_name>
    # will be used if defined
    # might be possible to define others

    # NOTE: ONLY set defaults on task attributes that are not inheritable,
    # inheritance is only triggered if the 'current value' is None,
    # default can be set at play/top level object and inheritance will take it's course.

    _args = FieldAttribute(isa='dict', default=dict)
    _action = FieldAttribute(isa='string')

    _async_val = FieldAttribute(isa='int', default=0, alias='async')
    _changed_when = FieldAttribute(isa='list', default=list)
    _delay = FieldAttribute(isa='int', default=5)
    _delegate_to = FieldAttribute(isa='string')
    _delegate_facts = FieldAttribute(isa='bool')
    _failed_when = FieldAttribute(isa='list', default=list)
    _loop = FieldAttribute()
    _loop_control = FieldAttribute(isa='class', class_type=LoopControl, inherit=False)
    _notify = FieldAttribute(isa='list')
    _poll = FieldAttribute(isa='int', default=C.DEFAULT_POLL_INTERVAL)
    _register = FieldAttribute(isa='string', static=True)
    _retries = FieldAttribute(isa='int', default=3)
    _until = FieldAttribute(isa='list', default=list)

    # deprecated, used to be loop and loop_args but loop has been repurposed
    _loop_with = FieldAttribute(isa='string', private=True, inherit=False)

```


```Handler Python class; attributes```

```xml
class Handler(Task):

    _listen = FieldAttribute(isa='list', default=list, listof=string_types, static=True)


```

### What is a ```Play``` ? 

A ```Play``` contains a set of ```task``` and ```Handlers``` for a set of ```hosts```.

### What is a ```task``` ?

A ```task``` is a set of instructions [using ```command```, ```shell```,```file```, ```apt```, ```yum```, ```debug``` or ```raw``` module] and reference of ```handler``` if needed.

### What is a ```Handler``` ?

A handler points to another  ```task``` that needs to be done or triggered after any change is done by a ```task```. Remember the word ```any change```. If nothing is changed no ```handler``` will be triggered. 

```Example of a play``` [example1](./examples/example1.yaml)

```xml
---

- hosts: Infra
  become: yes
  tasks:
    - name: install vsftpd on ubuntu
      apt: name=vsftpd update_cache=yes state=latest
      ignore_errors: yes
      notify: start vsftpd
  handlers:
     - name: start vsftpd
       service: name=vsftpd enabled=yes state=started

```
Here ```notify``` triggers a ```handler```. ```Service``` has three args, ```name``` name of the ```service```, ```enabled```, a boolean that states whether this ```service``` is going to start ```on boot time```, ```state```  as ```started``` means currrent expected ```state``` is ```started```.

To ```run``` this playbook,

``` ansible-playbook example1.yaml ```

## Ansible variables and facts

Declare variable like follwoing, [example](./examples/variable.yaml), 

```xml
---
- hosts: Infra
  vars: 
    - var1: hello
    - var2: world
  tasks:
    - name: echo hello world
      shell: echo "{{var1}} {{var2}} "
```
What are ```facts``` ?

```Facts``` are a wide range ```variables``` those can be referenced inside our ```playbooks```. 
```Fact``` inculdes,

- CPU type
- OS family
- RAM amount
- IP address
- CPU Core

and lot more.

To list all of facts run the following command,

```
ansible [goup name] -m setup
```
For example,

``` ansible Infra -m setup ```



## Ansible conditional

[Example](examples/conditionals.yaml),

```xml
---

- hosts: Infra
  become: yes
  tasks:
    - name: install apache2
      apt: name=apache2 state=latest
      ignore_errors: yes
      register: results

    - name: install httpd
      yum: name=httpd state=latest
      when: results|failed

```
In this example, we're trying to install apache2 on our Infra nodes. Suppose we've tow nodes in this group; a ```centos``` and an ```ubuntu```. As ```apt``` is not available in ```centos```, this node will return ```failed``` and if so happen then the second task will get triggered. 

Another way of doing this,[example](./examples/conditionals2.yaml)

```xml

---

- hosts: Infra
  become: yes
  tasks:
    - name: install apache2
      apt: name=apache2 state=latest
      when: ansible_os_family == "Debian"

    - name: install httpd
      yum: name=httpd state=latest
      when: ansible_os_family == "RedHat"
```
Here ```ansible_os_family``` is ```fact``.


## Ansible loops

Ansible supports many types of loops. Let's explore some,

[Standard loop](./examples/standard_loop.yaml)



```xml
---
- hosts: Infra
  become: yes
  tasks:
    - name: install stuff
      apt: name={{ item }} update_cache=yes state=latest
      with_items:
         - vim
         - name
         - apache2
```

```xml
---
- hosts: Infra
  become: yes
  tasks:
    - name: show contents
      debug: msg={{item}}
      with_file:
         - file1.txt
         - file2.txt

```

```xml
---
- hosts: Infra
  become: yes
  tasks:
    - name: print sequence
      debug: msg={{item}}
      with_sequence: star=1 end=10

```

## Some shortcuts

To wait until all the pods are running,

```xml
- name: wait for pods to come up
  shell: kubectl get pods -n name_of_namespace -o json
  register: kubectl_get_pods
  until: kubectl_get_pods.stdout|from_json|json_query('items[*].status.phase')|unique == ["Running"]
```
To generate string and set to variables or facts

```xml
- name: reset klovercloud_super_admin_username and klovercloud_super_admin_password
  set_fact:
    admin_username: "{{ lookup('password', '/dev/null length=8 chars=ascii_letters') }}"
```
To create a directory

```xml
- name: Creates /etc/kubernetes/kubeadm directory
  file: path=/etc/kubernetes/kubeadm/klovercloud/configs  recurse=yes state=directory
  when: >
    inventory_hostname == "kube-master-0"
```

To copy files from template to the directory

```xml

- name: Copy harbor installation, klovercloud-harbor and mongodb descriptors
  template: src="{{ item.src }}" dest="{{ item.dest }}"
  with_items:
     - { src: 'filename.yaml.j2', dest: '/etc/kubernetes/kubeadm/klovercloud/configs/filename.yaml'  } 
  when: >
    inventory_hostname == "kube-master-0"
  register: result

```
To apply to master only
```xml
- name:  
  shell: kubectl apply -f /etc/kubernetes/kubeadm/klovercloud/configs/filename.yaml;
  when: >
    inventory_hostname == "kube-master-0" 
```
Using k8s,

```xml
- name: deploy Kafka
  k8s:
   state: present
   definition: "{{ lookup('file', '/etc/kubernetes/kubeadm/klovercloud/configs/filename.yaml') }}"
   wait: yes
   wait_timeout: 300
   wait_condition:
    type: Complete
    status: True
```
Make curl inside pod
```xml
- name: Request to create a facade Service access token
  shell: kubectl -n {{ namespace }} exec -it $(kubectl -n {{ namespace }} get pod -l "app={{ label }}" -o jsonpath='{.items[0].metadata.name}') -- bin/sh -c "curl -X POST 'http://localhost:{{ port }}/api/api-access-token'  -H 'Content-Type:application/json' -H 'x-auth-token:{{ token }}' -d '{\"userId\":\"12\"}'"
  register: api_access_token

- name: reset api_access_token
  set_fact:
    api_access_token: "{{ api_access_token.stdout | from_json }}"
 ```

[learn more about loops](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)
