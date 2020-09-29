# Migrate GitLab to new Host

## Install GitLab 11.11.0 version

Current GitLab version is 11.11.0:

![image](./images/GitLabVersion.png)

We need to install the same version to server. Getlab can be installed from https://packages.gitlab.com/gitlab/gitlab-ce?page=71, additionally you  can take a look on official instruction to get information about Manually download and install a GitLab package:
https://docs.gitlab.com/omnibus/manual_install.html

```bash
wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/ubuntu/bionic/gitlab-ce_11.11.0-ce.0_amd64.deb/download.deb

# Install and  setup openssh-server
sudo apt-get install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh

# GitLab Community Edition
# Debian/Ubuntu
sudo dpkg -i gitlab-ce_11.11.0-ce.0_amd64.deb

```

You should see this output:

![image](./images/GitLabInstalled.png)

Next step - setup and first run Gitlab

```bash
sudo nano /etc/gitlab/gitlab.rb
```
Set external_url parameter to `'http://<hostname>:<port if needed>'`

![image](./images/SetExternalUrl.png)

Save changes and exit: Ctrl+S, Ctrl+X

Then we need to restart Gitlab by command:

```bash
sudo gitlab-ctl reconfigure
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

