# Secure Your Server

Steps to secure an Ubuntu 22.04 LTS server

- [Creating a new User](#creating-a-new-user)
- [Installing 2FA](#installing-2fa)
- [fail2ban](#fail2ban)
- [Firewall](#firewall)

***

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

***

## Firewall

Here's how you can secure your server with `ufw`.
Keep in mind that these are basic rules and you may need to adjust them based on your specific needs.

1. Check the current status of ufw by typing the following command:
   ```sh
   sudo ufw status
   ```

1. By default, ufw should be inactive. If it's not, you can disable it using the following command:
   ```sh
   sudo ufw disable
   ```

1. Set the default policy for incoming and outgoing traffic to deny all connections:
   ```sh
   sudo ufw default deny incoming
   sudo ufw default deny outgoing
   ```

1. Allow SSH connections so you can still access your server remotely. Use the following command to enable SSH:
   ```sh
   sudo ufw allow ssh
   ```

1. If you're running a web server, allow HTTP and HTTPS traffic by using the following commands:
   ```sh
   sudo ufw allow http
   sudo ufw allow https
   ```

1. If you're running other services, such as FTP or SMTP, you'll need to allow those ports as well. For example, to allow FTP traffic:
   ```sh
   sudo ufw allow ftp
   ```

1. After allowing the necessary traffic, enable ufw:
   ```sh
   sudo ufw enable
   ```

1. Check the status again to make sure ufw is running:
   ```sh
   sudo ufw status
   ```