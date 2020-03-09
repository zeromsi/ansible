# ansible

``` Ansible use ssh client to perform jobs to the nodes. Ansible is agent less. That is why ansible needs to installed on the controll machine only.```

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

## Our ansible controll machine must have the privileges to log in to the nodes and have the sudo privileges. We don't want to remember the IP address of the nodes as this is very difficult. So what we can do, we can provide a domain name to the nodes. To do this, we need to edit hosts file of our controll machine.

``` 
vi /ect/hosts

```

