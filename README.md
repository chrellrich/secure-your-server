# Secure Your Server

Steps to secure an Ubuntu 22.04 LTS server

- [Creating a new User](#creating-a-new-user)
- [Installing 2FA](#installing-2fa)
- [fail2ban](#fail2ban)




## Creating a new User

Create a new user, so you don't use root to log in.

```sh
adduser <user>
usermod -aG sudo
```

***

## Installing 2FA

Two-factor authentication (2FA) is a security process that requires users to provide two different forms of identification to access the system.
By requiring two factors, 2FA adds a layer of security beyond just a password, making it more difficult for unauthorized users to gain access to sensitive information or systems.

### Install the Google Authenticator PAM Module

```sh
sudo apt install libpam-google-authenticator
```

### Run the Initialization

Run `google-authenticator -t -d -f -r 3 -R 30 -w 3` to automatically set the following options:

- t => Time based counter
- d => Disallow token reuse
- f => Force writing the settings to file without prompting the user
- r => How many attempts to enter the correct code
- R => How long in seconds a user can attempt to enter the correct code
- w => How many codes can are valid at a time (this references the 1:30 min)

To choose other Options run `google-authenticator` without any parameters.

Scan the QR-Code or copy the 2FA code.

### Configuring SSH

#### /etc/pam.d/sshd

In `/etc/pam.d/sshd` add `auth required pam_google_authenticator.so` as shown
```
# Standard Un*x authentication.
@include common-auth
auth required pam_google_authenticator.so
```

#### /etc/ssh/sshd_config

In `/etc/ssh/sshd_config` set `KbdInteractiveAuthentication` to `yes`
```
# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
KbdInteractiveAuthentication yes
```

Disable password authentication, rootlogin and only allow selected user(s) at the bottom
```
PasswordAuthentication no
PermitRootLogin no
AllowUsers <user>
```

Allow authentication via pubickey or password and 2fa code with pam (needs to be added at the bottom)
```
AuthenticationMethods publickey keyboard-interactive
```

### Restart SSH

```sh
sudo systemctl restart sshd
```

***

## fail2ban

fail2ban is a security tool that monitors log files for suspicious activity on servers,
such as multiple failed login attempts, and automatically blocks access from the offending IP address

### Installation

```sh
sudo apt install fail2ban
```

### Changing the Bantime

Copy the config file

```sh
sudo cp /etc/fail2ban/jail.{conf,local}
```

Change bantime to one day in `/etc/fail2ban/jail.local`:

```
bantime  = 1d
```

### Start the Service.

```sh
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```


### fail2ban-client

#### Status of sshd Jail

```sh
sudo fail2ban-client status sshd
```

#### Unban an IP

```sh
sudo fail2ban-client set sshd unbanip 23.34.45.56
```
