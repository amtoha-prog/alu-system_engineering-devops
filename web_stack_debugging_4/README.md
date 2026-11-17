# Web Stack Debugging 4

This project fixes an nginx web server that fails under heavy load.

## Problem

Benchmarking with ApacheBench (`ab -c 100 -n 2000 localhost/`) showed many failed requests. The cause is nginx's open-file limit (`ULIMIT` in `/etc/default/nginx`), which was too low to handle 100 concurrent connections.

## Files

| File | Description |
|------|-------------|
| `0-the_sky_is_the_limit_not.pp` | Puppet manifest that raises nginx's `ULIMIT` to `-n 4096` and restarts nginx |

## Usage

```bash
puppet apply 0-the_sky_is_the_limit_not.pp
ab -c 100 -n 2000 localhost/
```

After applying the manifest, `Failed requests` should be `0`.
