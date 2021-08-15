# SSH Server Setup
The followings step are based on Manjaro OS. Using your OS package manager you could install all the required tools without issues.
All the configurations are based on tool (openssh) and it should be valid in each OS

## Install and run OpenSSH
Update your system

```sh
sudo pacman -Syu
```

Install [openssh](openssh.com)

```sh
sudo pacman -S openssh
```

After openssh is correctly installed you can check the status of the service

```sh
sudo systemctl status sshd
```

> If the Open SSH server isn’t running, the terminal should say “inactive”. If that’s the case, you may run Open SSH by entering the following command:
> ```sh
> sudo systemctl start sshd
> ```

If Open SSH is running, the prompt will say “active” in green.
If you want to terminate the SSH server, type in the following:

```sh
sudo systemctl stop sshd
```

If you have changed the configuration of your ssh server you need to reboot the service to apply them

```sh
sudo systemctl restart sshd
```


### Automate SSH server startup
To automatically start the SSH server upon system reboot, you can use enter the following code:

```sh
sudo systemctl enable sshd
```

> You can remove Open SSH from the system startup with the following command:
> ```sh
> sudo systemctl disable sshd
> ```

## Configuration
You can locate the Open SSH server files in the following location on your hard drive and you can edit it to customise the settings

```sh
sudo vim /etc/ssh/sshd_config
```

> To obtain a list of all the options available that we can configure, type in the following code:
> ```sh
> man sshd_config
> ```

### Improve server security 
You can see that the default port SSH server listens to is port 22. This port is easily "victim" of malicious scanning software. It is strongly suggested to change it with any value of your choice and you can ue any unprivileged port between 1024 and 65535

> When you connect to the server you can specify the port using the flag **-p**
> ```sh
> ssh -p PORT_NUMBER USERNAME@IP_ADDRESS
> ```

In 2006, the SSH protocol was updated from version 1 to version 2. It was a significant upgrade. There were so many changes and improvements, especially around encryption and security, that version 2 is not backward compatible with version 1. To prevent connections from version 1 clients, you can stipulate that your computer will only accept connections from version 2 clients. To do so add in the configuration file the line:
```
Protocol 2
```

> To connect to an SSH server that is only supporting the new protocol you need to pass the **-2** flag
> ```sh
> ssh -2 USERNAME@IP_ADDRESS
> ```

Although it is a bad practice, a Linux system administrator can create a user account with no password. That means remote connection requests from that account will have no password to check against. Those connections will be accepted but unauthenticated.
To ensure all connections are authenticated we can set the option:

```sh
PermitEmptyPasswords no
```

SSH keys provide a secure means of logging into an SSH server. Passwords can be guessed, cracked, or brute-forced. SSH keys are not open to such types of attack.

> Of course, the logical extension of using SSH keys is that if all remote users are forced to adopt them, you can turn off password authentication completely. To do so, edit your SSH configuration file:
> ```sh
> PasswordAuthentication no
> ```
> By default the public key authentication is enable but to set it explicitely use the configuratin:
> ```sh
> PubkeyAuthentication yes
> ```

X11 forwarding allows remote users to run graphical applications from your server over an SSH session. In the hands of a threat actor or malicious user, a GUI interface can make their malign purposes easier.

> A standard mantra in cybersecurity is if you don’t have a bonafide reason to have it turned on, turn it off. To disable it edit your SSH configuration file:
> ```sh
> X11Forwarding no
> ```

If there is an established SSH connection to your computer, and there has been no activity on it for a period of time, it could pose a security risk. There is a chance that the user has left their desk and is busy elsewhere. Anyone else who passes by their desk can sit down and start using their computer and, via SSH, your computer.

> It’s much safer to establish a timeout limit. The SSH connection will be dropped if the inactive period matches the time limit. To set it edit your SSH configuration file:
> ```sh
> ClientAliveInterval 300
> ```
> The value passed to the option is in seconds (300 seconds = 5 minutes)

Defining a limit on the number of authentication attempts can help thwart password guessing and brute-force attacks. After the designated number of authentication requests, the user will be disconnected from the SSH server.

> By default, there is no limit. To set it edit your SSH configuration file:
> ```sh
> MaxAuthTries 3
> ```

It is bad practice to log in as root on your Linux computer. You should log in as a normal user and use sudo to perform actions that require root privileges. Even more so, you shouldn’t allow root to log into your SSH server.

> By default the root user login is allowed. To set it edit your SSH configuration file:
> ```sh
> PermitRootLogin no
> ```

To provide another layer of security, you should limit your SSH logins to only certain users who need remote access. This way, you will minimize the impact of having a user with a weak password.

> Open your /etc/ssh/sshd_config file to add an ‘AllowUsers’ line, followed by the list of usernames, and separate them with a space:
> ```sh
> AllowUsers user1 user2
> ```

To apply som of the configuration only for a specific user. You can override the default settings using a `match section` in the endo of the cofgiguration file:

```sh
Match User username
    X11Farwarding yes
    AllowTcpForwarding yes
    ...
```

Of course, if you don’t need SSH running on your computer at all and you want to have a high level of security, make sure it is disabled.

```sh
sudo systemctl stop sshd
```
```sh
sudo systemctl disable sshd
```

## SSH Keys Authentication
An SSH server can authenticate clients using a variety of different methods. The most basic of these is password authentication, which is easy to use, but not the most secure.

Although passwords are sent to the server in a secure manner, they are generally not complex or long enough to be resistant to repeated, persistent attackers. SSH keys prove to be a reliable and secure alternative.

> SSH key pairs are two cryptographically secure keys that can be used to authenticate a client to an SSH server. Each key pair consists of a public key and a private key.

The public key is uploaded to a remote server that you want to be able to log into with SSH. The key is added to a special file within the user account you will be logging into called `~/.ssh/authorized_keys`.

When a client attempts to authenticate using SSH keys, the server can test the client on whether they are in possession of the private key. If the client can prove that it owns the private key, a shell session is spawned or the requested command is executed.

### Create SSH Key
The first step to configure SSH key authentication to your server is to generate an SSH key pair on your local computer.

On your local computer, generate a SSH key pair by typing:

```sh
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

> By default, the keys will be stored in the `~/.ssh` directory within your user’s home directory. The private key will be called `id_rsa` and the associated public key will be called `id_rsa.pub`.

### Add SSH key to ssh-agent
Start the ssh-agent in the background.

```sh
eval "$(ssh-agent -s)"
```

Add your SSH private key to the `ssh-agent`.

```sh
ssh-add ~/.ssh/PRIVATE_KEY_FILE_NAME
```

### Copying your Public Key Using SSH
Allowing the password-based SSH access to an account on your server, you can upload your keys using a conventional SSH method.
We can do this by outputting the content of our public SSH key on our local computer and piping it through an SSH connection to the remote server.

```sh
cat ~/.ssh/PUBLIC_KEY.pub | ssh USERNAME@IP_ADDRESS "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

> We use the `>>` redirect symbol to append the content instead of overwriting it. This will let us add keys without destroying previously added keys.

After entering your password, the content of your `PUBLIC_KEY.pub` key will be copied to the end of the `authorized_keys` file of the remote user’s account.

> If you have successfully completed one of the procedures above, you should be able to log into the remote host without the remote account’s password.
> ```sh
> ssh USERNAME@IP_ADDRESS
> ```

## SSH Tunneling
SSH port forwarding is a mechanism in SSH for tunneling application ports from the client machine to the server machine, or vice versa. 

In OpenSSH, remote SSH port forwardings are specified using the `-R` option. For example:

```sh
ssh -R 8080:localhost:80 USERNAME@IP_ADDRESS
```

By default, OpenSSH only allows connecting to remote forwarded ports from the server host. However, the `GatewayPorts` option in the server configuration file `sshd_config` can be used to control this.

```sh
GatewayPorts clientspecified
```

This means that the client can specify an IP address from which connections to the port are allowed. The syntax for this is:

```sh
ssh -R 52.194.1.73:8080:localhost:80 host147.aws.example.com
```

In this example, only connections from the IP address 52.194.1.73 to port 8080 are allowed.

> OpenSSH also allows the forwarded remote port to specified as 0. In this case, the server will dynamically allocate a port and report it to the client. When used with the -O forward option, the client will print the allocated port number to standard output.


## Improve security with Firewall
If you want your server to be reachable from only a specific IP address on port 22, then you should consider filtering connections at your firewall by adding a firewall rule on your router or update your iptables like this:

```sh
iptables -A INPUT -p tcp -s YourIP --dport 22 -j ACCEPT
```

> With that rule, you are opening the SSH port only to YourIP.

If it is essential for you to open the SSH port globally, then iptables can still help prevent heavy-handed attacks by logging and blocking repeated attempts to login from the same IP address.
The following rule records the IP address of each new attempt to access port 22:

```sh
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set --name ssh –rsource
```

The following rule verifies if that IP address has tried to connect three times or more within the last 90 seconds. If it hasn’t, then the packet is accepted (this rule would need a default policy of DROP on the input chain).

```sh
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent ! --rcheck --seconds 90 --hitcount 3 --name ssh --rsource -j ACCEPT
```

Filtering at the firewall is a tremendously useful method of securing access to your SSH server.

### Uncomplicated Firewall
```sh
sudo pamac install ufw
```

Once UFW is installed you need to start and enable it using the commands: 

```sh
sudo systemctl enable ufw.service
sudo ufw enable
```

Our first method is the most complicated--simply because it makes use of iptables, by way of UFW. Let's say the offending IP address you want to block is 192.168.1.162. We're going to block this using UFW, with a single command:

```ssh
sudo ufw deny from 192.168.1.162 port 22
```

> Check to see if UWF is running and enabled with the command
> ```sh
> sudo uwf status
> ```


## Router configuration

### Find out Gateway / router IP address
You need to use the router command command. This command can manipulate the kernel’s IP routing tables. It can also be used to print gateway/router IP address. Type the following command to see default gateway.

```sh
route -n 
```

> The flag U indicates that route is up and G indicates that it is gateway.
> The router ip is the one in the Gateway column.


Find router ip
```sh
ip route list | grep default
```

### Router configs
Access the router configuration console with the model default credentials if you haven't updated them.
Follow the manual for the right steps based on the router brand and model.

Find MAC and ip address of your device


Find decice ip
```sh
ip addr show
```

> The hostname will be the name of the device


Public router ip

```sh
curl https://ipinfo.io/ip
```





----

# Future steps

## TWO Factor Authentication
> https://www.linux.com/training-tutorials/securing-ssh-two-factor-authentication-using-google-authenticator/

## Access logging
Options:
```sh
SyslogFacility AUTH
LogLevel INFO
```

## Client Key Farwarding


    Create ~/.ssh/config

    Fill it with (host address is the address of the host you want to allow creds to be forwarded to):

    Host [host address]
         ForwardAgent yes

    If you haven't already run ssh-agent, run it:

    ssh-agent

    Take the output from that command and paste it into the terminal. This will set the environment variables that need to be set for agent forwarding to work. Optionally, you can replace this and step 3 with:

    eval "$(ssh-agent)"

    Add the key you want forwarded to the ssh agent:

    ssh-add [path to key if there is one]/[key_name].pem

    Log into the remote host:

    ssh -A [user]@[hostname]

    From here, if you log into another host that accepts that key, it will just work:

    ssh [user]@[hostname]

