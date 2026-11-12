# web_stack_debugging_3
 
Web stack debugging project: find out why Apache returns a 500 error, fix it by hand, then automate the fix with Puppet instead of Bash.
 
**Author:** amtoha-prog
 
## Concepts
 
- **HTTP 500 error**: the server failed while handling the request. The cause is usually visible in the server or application logs.
- **strace**: records the system calls a program makes. A failed file lookup shows up as `ENOENT` (no such file or directory).
- **Apache error log**: `/var/log/apache2/error.log` names the file and line behind a PHP error, which is often easier to read than a strace output.
- **Puppet**: a configuration tool. A manifest (`.pp` file) describes the state you want, and `puppet apply` makes the machine match it.
- **`exec` resource**: a Puppet resource that runs a command.
- **puppet-lint**: checks that a manifest follows the Puppet style guide.
## Files
 
| File | Task | Description |
|------|------|-------------|
| `0-strace_is_your_friend.pp` | 0 | Puppet manifest that fixes the Apache 500 error |
 
## Task 0: Strace is your friend
 
**Problem:** Apache returns `HTTP/1.0 500 Internal Server Error` for a WordPress site.
 
**Investigation** (inside the container):
 
```bash
curl -sI 127.0.0.1                         # reproduce the 500 error
tail -20 /var/log/apache2/error.log        # which file does PHP fail to open?
grep -rn "phpp" /var/www/html/             # which file contains the wrong name?
```
 
To use strace, start Apache under it (attaching to a running process needs the `SYS_PTRACE` capability, which containers usually do not have):
 
```bash
service apache2 stop
. /etc/apache2/envvars
strace -f -s 300 -o /tmp/trace.txt /usr/sbin/apache2 -X &
curl -sI 127.0.0.1
pkill apache2
grep -n "ENOENT" /tmp/trace.txt | grep -v -E "\.so|/proc|/sys" | tail
```
 
**Cause found:** `/var/www/html/wp-settings.php` requires a file whose name ends in `.phpp` instead of `.php`. The file does not exist, so PHP stops with a fatal error and Apache returns a 500.
 
**Fix:** correct the extension in `wp-settings.php`.
 
**Manifest** (`0-strace_is_your_friend.pp`):
 
```puppet
# Fixes the Apache 500 error caused by a misspelled file extension
exec { 'fix-wordpress':
  command => 'sed -i s/phpp/php/g /var/www/html/wp-settings.php',
  path    => '/bin:/usr/bin',
}
```
 
The command is safe to run more than once: once the typo is fixed, `sed` changes nothing.
 
## Usage
 
```bash
curl -sI 127.0.0.1
puppet apply 0-strace_is_your_friend.pp
curl -sI 127.0.0.1
puppet-lint 0-strace_is_your_friend.pp
```
 
Expected result: the first `curl` shows `HTTP/1.1 500` or `HTTP/1.0 500`, Puppet reports `Exec[fix-wordpress]/returns: executed successfully`, the second `curl` shows `HTTP/1.1 200 OK`, and `puppet-lint` prints nothing.
 
Ruby may print warnings such as `URI.escape is obsolete` when Puppet runs. They come from old Puppet code on a newer Ruby and do not affect the result.
