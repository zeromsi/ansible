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






