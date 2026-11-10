# web_stack_debugging_0
 
Web stack debugging project: find out why a broken web server container does not work, fix it by hand, then write a Bash script that applies the fix.
 
**Author:** amtoha-prog
 
## Concepts
 
- **Debugging a web stack**: asking simple questions in order. Is the program installed? Is it running? Does it serve the right content?
- **Docker container**: an isolated environment used to run the broken server.
- **Port mapping**: `-p 8090:80` connects port 8090 on the host to port 80 inside the container.
- **Empty reply from server**: the port is open, but no program behind it answers.
## Files
 
| File | Task | Description |
|------|------|-------------|
| `0-give_me_a_page` | 0 | Bash script that fixes the container so Apache returns a page containing `Hello Holberton` |
 
## Task 0: Give me a page!
 
**Problem:** the container `holbertonschool/265-0` has Apache installed and a page containing `Hello Holberton`, but querying port 80 returns an error such as `Empty reply from server`.
 
**Investigation:**
 
1. `service apache2 status` showed that Apache was **not running**.
2. `/var/www/html/index.html` already contained `Hello Holberton`.
**Fix:** start Apache.
 
```bash
service apache2 start
```
 
## Usage
 
Start the broken container:
 
```bash
docker run -p 8091:80 -d -it holbertonschool/265-0
curl localhost:8091
```
 
Expected result: the request fails (`Empty reply from server` or `Connection reset by peer`).
 
Run the script inside the container (replace `<container id>` with the real ID from `docker ps`):
 
```bash
docker exec -i <container id> bash < 0-give_me_a_page
curl localhost:8091
```
 
Expected result:
 
```
Hello Holberton
```
