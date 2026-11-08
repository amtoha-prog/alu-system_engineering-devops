# Web Server

This project covers setting up and configuring a web server on an Ubuntu 20.04 machine.

## Files

| File | Description |
|------|-------------|
| `0-transfer_file` | Bash script that transfers a file to a server's home directory with `scp`. Usage: `./0-transfer_file PATH_TO_FILE IP USERNAME PATH_TO_SSH_KEY` |
| `1-install_nginx_web_server` | Installs nginx and serves a page containing `Holberton School` on port 80 |
| `2-setup_a_domain_name` | Contains the .tech domain name pointing to the web-01 server |
| `3-redirection` | Configures nginx with a 301 redirect from `/redirect_me` |
| `4-not_found_page_404` | Configures nginx with a custom 404 page containing `Ceci n'est pas une page` |
| `5-design_a_beautiful_404_page.html` | A custom designed 404 page |

## Requirements

- Ubuntu 20.04 LTS
- All scripts start with `#!/usr/bin/env bash` and are executable
