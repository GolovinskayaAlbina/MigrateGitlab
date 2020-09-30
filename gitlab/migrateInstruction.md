# Migrate GitLab to new Host

## Install GitLab 11.11.0 version

Current GitLab version is 11.11.0:

![image](./images/GitLabVersion.png)

We need to install the same version to server. Getlab can be installed from https://packages.gitlab.com/gitlab/gitlab-ce?page=71, additionally you  can take a look on official instruction to get information about Manually download and install a GitLab package:
https://docs.gitlab.com/omnibus/manual_install.html

```bash
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_11.11.0-ce.0_amd64.deb/download.deb

# root
sudo su

# Install and  setup openssh-server
apt-get install openssh-server
systemctl enable ssh
systemctl start ssh

# GitLab Community Edition
# Debian/Ubuntu
dpkg -i gitlab-ce_11.11.0-ce.0_amd64.deb

```

You should see this output:

![image](./images/GitLabInstalled.png)

Next step - setup and first run Gitlab

```bash
nano /etc/gitlab/gitlab.rb
```
Set external_url parameter to `'http://<hostname>:<port if needed>'`

![image](./images/SetExternalUrl.png)

Save changes and exit: Ctrl+S, Ctrl+X

Then we need to restart Gitlab by command:

```bash
gitlab-ctl reconfigure
```

Go to your site, in my case it is  http://localhost:8084/ and set `root` password, then sing in as root user


![image](./images/FirstGitLabOpen.png)


Check help page. The version should be the same

![image](./images/HelpPageVersion.png)

## Migrate projects

### Creating backups

#### Backup GitLab directory

Go to source GitLab server and run:

```bash
sudo su
cd ~
mkdir GiLabMigrate
cd GiLabMigrate
sudo sh -c 'umask 0077; tar -cf $(date "+etc-gitlab-%s.tar") -C / etc/gitlab'
ls
```

This creates the following file in the current working directory.

```bash		
ls -la etc-gitlab-1601386671.tar
```

> -rw------- 1 root root 102400 May 16 19:47 etc-gitlab-1601386671.tar

With the following content.

```bash	
tar -tvf etc-gitlab-1601386671.tar
```


> drwxrwxr-x root/root         0 2020-09-29 13:46 etc/gitlab/

> -rw------- root/root     91815 2020-09-29 13:42 etc/gitlab/gitlab.rb

> -rw------- root/root     15370 2020-09-29 13:42 etc/gitlab/gitlab-secrets.json

> drwxr-xr-x root/root         0 2020-09-29 13:32 etc/gitlab/trusted-certs/


#### Application Backup

By default, backups are kept in /var/opt/gitlab/backups. Check this setting

```bash	
grep backup_path /etc/gitlab/gitlab.rb
```
> gitlab_rails['manage_backup_path'] = true

> gitlab_rails['backup_path'] = "/var/opt/gitlab/backups"


If you change this location, be sure to run the below after.

```bash	
gitlab-ctl reconfigure
```

To actually take the application backup, run the following.

```bash
gitlab-rake gitlab:backup:create
```
This creates a backup in the following location.
	
```bash
ls -ltr /var/opt/gitlab/backups/
```
>total 1020

>-rw------- 1 git git 1044480 сен 29 16:54 1601387650_2020_09_29_11.11.0_gitlab_backup.tar

#### Prepare Zip

Lets zip and compress these files into a single archive for transfer. 
	
> /var/opt/gitlab/backups/1601387650_2020_09_29_11.11.0_gitlab_backup.tar

> ~/GiLabMigrate/etc-gitlab-1601386671.tar

Zip and compress with the following.

```bash
cp 	/var/opt/gitlab/backups/1601387650_2020_09_29_11.11.0_gitlab_backup.tar ~/GiLabMigrate/
tar -cvzf ~/gitlab-backup.tar.gz ~/GiLabMigrate/
```

## Application Backup Restore

Stop processes using the database.


```bash
sudo su	
gitlab-ctl stop unicorn
gitlab-ctl stop sidekiq
gitlab-ctl status
```

Check they’re actually down.

```bash	
gitlab-ctl status | grep ^down
```

Move the application backup to the backups directory as defined in gitlab.rb and correct the owner if necessary.

```bash	
cp -v ~/GiLabMigrate/1601387650_2020_09_29_11.11.0_gitlab_backup.tar /var/opt/gitlab/backups/
chown -v git:git /var/opt/gitlab/backups/1601387650_2020_09_29_11.11.0_gitlab_backup.tar
```

Then restore. The restore name is the same as the above file name but without the _gitlab_backup.tar bit.

```bash		
gitlab-rake gitlab:backup:restore BACKUP=1601387650_2020_09_29_11.11.0
```	

Now restart GitLab.

```bash		
gitlab-ctl start
```

And check for problems.

```bash		
gitlab-rake gitlab:check SANITIZE=true
```

If all was successful, you should be able to log into the new site as before with whatever users you had setup. If you had 2FA enabled, you may need to restore the gitlab-secrets.json. Fro more information take a look on this [link](https://pikedom.com/migrate-gitlab-instance-to-new-host/)

## Update GitLab version 

```bash
#optionally. New version will update it
sudo gitlab-ctl pg-upgrade

sudo gitlab-rake gitlab:env:info
```

![image](./images/UpgradePG.png)

You can run sudo gitlab-ctl pg-upgrade command one more time if version of DB in not 10.7

```bash
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ee/packages/ubuntu/bionic/gitlab-ee_12.0.6-ee.0_amd64.deb/download.deb

sudo dpkg -i gitlab-ee_12.0.6-ee.0_amd64.deb 
```

![image](./images/GitLabVers12.0.png)


Then update version to 12.10.9

```bash
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ee/packages/ubuntu/bionic/gitlab-ee_12.10.9-ee.0_amd64.deb/download.deb

sudo dpkg -i gitlab-ee_12.10.9-ee.0_amd64.deb 
```

For more information about PostgreSql and GitLab click the [link](https://docs.gitlab.com/omnibus/package-information/postgresql_versions.html) 

![image](./images/GitLabVer12.10.png)

Then update to 13.0.x version
```bash
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ee/packages/ubuntu/bionic/gitlab-ee_13.0.9-ee.0_amd64.deb/download.deb

sudo dpkg -i gitlab-ee_13.0.9-ee.0_amd64.deb 
```

Plese, skip 13.4.1 updating. It removes migrated projects.
{: .alert .alert-warning}


Las step - updating to the latest 13.4.1 version



```bash
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ee/packages/ubuntu/bionic/gitlab-ee_13.4.1-ee.0_amd64.deb/download.deb

sudo dpkg -i gitlab-ee_13.4.1-ee.0_amd64.deb 

sudo gitlab-ctl status
```
Output should looks like 

```bash
#OUTOUT
run: alertmanager: (pid 230469) 12s; run: log: (pid 230383) 32s
run: gitaly: (pid 230459) 12s; run: log: (pid 78050) 6278s
run: gitlab-exporter: (pid 230484) 12s; run: log: (pid 230374) 35s
run: gitlab-workhorse: (pid 230412) 14s; run: log: (pid 230348) 38s
run: grafana: (pid 230448) 13s; run: log: (pid 230389) 31s
run: logrotate: (pid 230488) 11s; run: log: (pid 230364) 36s
run: nginx: (pid 230498) 11s; run: log: (pid 230361) 37s
run: node-exporter: (pid 230508) 10s; run: log: (pid 230370) 35s
run: postgres-exporter: (pid 230429) 13s; run: log: (pid 230386) 32s
run: postgresql: (pid 202936) 1191s; run: log: (pid 38750) 8328s
run: prometheus: (pid 230593) 10s; run: log: (pid 230380) 33s
run: puma: (pid 230605) 10s; run: log: (pid 230342) 39s
run: redis: (pid 218178) 418s; run: log: (pid 38735) 8328s
run: redis-exporter: (pid 230611) 9s; run: log: (pid 230377) 34s
run: sidekiq: (pid 230618) 9s; run: log: (pid 230345) 39s
```
![image](./images/GitLabVer13.4.png)

## Remove GitLab

![image](./images/RemoveGitLab.png)