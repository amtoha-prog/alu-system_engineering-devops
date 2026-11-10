# web_stack_debugging_1
 
Web stack debugging project: find out why Nginx is not listening on port 80 in an Ubuntu container, fix it by hand, then automate the fix in a Bash script.
 
**Author:** amtoha-prog
 
## Concepts
 
- **Debugging a web stack**: ask simple questions in order. Is the program installed? Is it running? Is it listening on the right port?
- **Listening port**: a program can be running without answering on a port. `ss -tlnp` shows which ports are open and which program owns them.
- **`sites-available` and `sites-enabled`**: Nginx only loads the sites linked in `sites-enabled`. `sites-available` is a storage folder, and a symbolic link in `sites-enabled` turns a site on.
- **`nginx -t`**: checks the configuration for syntax errors. A valid configuration can still be useless if no site is enabled.
- **`0.0.0.0:80`**: listening on port 80 on all IPv4 addresses of the server.
## Files
 
| File | Task | Description |
|------|------|-------------|
| `0-nginx_likes_port_80` | 0 | Bash script that makes Nginx run and listen on port 80 on all active IPv4 addresses |
 
## Task 0: Nginx likes port 80
 
**Problem:** Nginx is installed in an Ubuntu container, but `curl 0:80` returns `Connection refused`.
 
**Investigation** (inside the container):
 
```bash
service nginx status                         # is it running?
ss -tlnp | grep ':80'                        # is anything listening on port 80?
nginx -t                                     # is the configuration valid?
ls -l /etc/nginx/sites-enabled/              # is a site enabled?
grep -Rn "listen" /etc/nginx/sites-enabled/  # which port does the site use?
```
 
**Causes reproduced and fixed:**
 
| Cause | What it looks like | Fix |
|-------|--------------------|-----|
| No site enabled | Nginx is running, `sites-enabled` is empty, nothing listens on port 80 | Link `default` into `sites-enabled` |
| Wrong port | `ss` shows Nginx listening on `0.0.0.0:8080` | Change `listen 8080` back to `listen 80` |
 
**Script** (`0-nginx_likes_port_80`):
 
```bash
#!/usr/bin/env bash
# Configures nginx to listen on port 80 on all active IPv4 addresses
ln -sf /etc/nginx/sites-available/default /etc/nginx/sites-enabled/default
sed -i 's/8080/80/g' /etc/nginx/sites-available/default
service nginx restart
```
 
- `ln -sf` creates the link, and replaces it if it already exists, so the script is safe to run again.
- `sed` changes the port only if `8080` is present.
- `service nginx restart` comes last so Nginx re-reads its configuration, and it also starts Nginx if it was stopped.
## Usage
 
Inside the container:
 
```bash
curl 0:80
./0-nginx_likes_port_80 > /dev/null 2>&1
curl 0:80
```
 
Expected result: the first `curl` fails with `Connection refused`, and the second returns the `Welcome to nginx!` page.
 
To confirm that Nginx listens on all IPv4 addresses:
 
```bash
ss -tlnp | grep ':80'
```
 
Expected result: a line containing `0.0.0.0:80` and `nginx`.
