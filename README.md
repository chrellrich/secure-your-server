# secure-your-server
Steps to secure a ubuntu 22.04 LTS server

## create new so you don't use root to login
```
adduser <user>
usermod -aG sudo
```

## install 2fa
```
sudo apt install libpam-google-authenticator
```

as the user run `google-authenticator`
scan qr code or copy 2fa code


in /etc/pam.d/sshd
add `auth required pam_google_authenticator.so` under 
# Standard Un*x authentication.
@include common-auth

in /etc/ssh/sshd_config
set KbdInteractiveAuthentication to yes

disable password authentication, rootlogin and only allow selected users
```
PasswordAuthentication no
PermitRootLogin no
AllowUsers <user>
```

Allow authentication via pubickey or password and 2fa code with pam
```
AuthenticationMethods publickey keyboard-interactive
```

## `sudo systemctl restart sshd` after changing config in sshd_config or pam.d/sshd

## install fail2ban
```
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.{conf,local}
sudo nano /etc/fail2ban/jail.local
sudo systemctl enable fail2ban
```

change bantime to `bantime  = 1d`

### fail2ban-client
`sudo fail2ban-client status sshd` show status of sshd jail
`sudo fail2ban-client set sshd unbanip 23.34.45.56` unban ip
