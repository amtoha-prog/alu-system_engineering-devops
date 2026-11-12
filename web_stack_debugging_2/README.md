# web_stack_debugging_2
 
Web stack debugging project: run software as a less privileged user instead of `root`, and fix a container so that Nginx runs as the `nginx` user.
 
**Author:** amtoha-prog
 
## Concepts
 
- **root**: the Linux superuser. It can do anything, so running services as `root` is risky. If an attacker breaks into a root process, they control the whole machine.
- **Least privilege**: run each service as a user that has only the permissions it needs.
- **Running a command as another user**: `sudo -u <user> <command>`, or `su -s /bin/bash <user> -c "<command>"` for users without a login shell.
- **Privileged ports**: only `root` can listen on ports below 1024, so a non-root web server uses a higher port such as `8080`.
- **Nginx processes**: the `user` directive in `nginx.conf` sets the user of the **worker** processes. The **master** process runs as whoever starts Nginx.
- **PID file**: Nginx writes its process ID to `/run/nginx.pid`. A non-root user needs permission to write it.
## Files
 
| File | Task | Description |
|------|------|-------------|
| `0-iamsomeoneelse` | 0 | Bash script that runs `whoami` under the user passed as an argument |
| `1-run_nginx_as_nginx` | 1 | Bash script that makes Nginx run as the `nginx` user and listen on port `8080` |
 
## Task 0: Run software as another user
 
The script accepts one argument and runs `whoami` as that user.
 
```bash
#!/usr/bin/env bash
# Runs the whoami command under the user passed as an argument
sudo -u "$1" whoami
```
 
**Usage**
 
```bash
root@ubuntu:~# whoami
root
root@ubuntu:~# ./0-iamsomeoneelse www-data
www-data
root@ubuntu:~# ./0-iamsomeoneelse nobody
nobody
root@ubuntu:~# whoami
root
```
 
## Task 1: Run Nginx as Nginx
 
**Problem:** Nginx runs as `root` (master process) and `www-data` (workers). It must run as `nginx` and listen on port `8080`.
 
**Investigation:**
 
```bash
ps auxff | grep ngin[x]                     # which user owns the processes?
grep -n "^user" /etc/nginx/nginx.conf       # which user do the workers use?
grep -Rn "listen" /etc/nginx/sites-enabled/ # which port is used?
ls -ld /var/log/nginx /var/lib/nginx        # who owns the folders Nginx writes to?
su -s /bin/bash nginx -c "/usr/sbin/nginx -t"  # does Nginx start as nginx?
```
 
**What had to be fixed:**
 
| Problem | Fix |
|---------|-----|
| Workers run as `www-data` | Set `user nginx;` in `nginx.conf` |
| Nginx listens on port `80` | Change `listen 80` to `listen 8080` in the site file |
| `nginx` cannot write logs and temp files | `chown -R nginx:nginx /var/log/nginx /var/lib/nginx` |
| `nginx` cannot create `/run/nginx.pid` (`Permission denied`) | Create the file and give it to `nginx` |
| Master process runs as `root` | Start Nginx as the `nginx` user with `su` |
 
**Script** (`1-run_nginx_as_nginx`):
 
```bash
#!/usr/bin/env bash
# Runs nginx as the nginx user and listening on port 8080
pkill -x nginx
sleep 1
sed -i 's/^user .*;/user nginx;/' /etc/nginx/nginx.conf
sed -i 's/\(listen.*[ :]\)80 default_server/\18080 default_server/' /etc/nginx/sites-available/default
chown -R nginx:nginx /var/log/nginx /var/lib/nginx
touch /run/nginx.pid
chown nginx:nginx /run/nginx.pid
su -s /bin/bash nginx -c "/usr/sbin/nginx"
```
 
Starting Nginx with `su` is the last command, so the master process belongs to `nginx`. `apt-get remove` is not used.
 
**Verify**
 
```bash
ps auxff | grep ngin[x]
nc -z 0 8080 ; echo $?
```
 
Expected result: every Nginx process, including the master, is owned by `nginx`, and `nc` prints `0`.
 
The warning `the "user" directive makes sense only if the master process runs with super-user privileges, ignored` is expected. It means the master is not running as `root`.
