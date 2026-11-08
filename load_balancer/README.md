# Load Balancer

This project adds redundancy to the web stack by running two nginx web servers behind an HAProxy load balancer on Ubuntu.

## Servers

| Name | Role |
|------|------|
| `7345-web-01` | nginx web server |
| `7345-web-02` | nginx web server (identical to web-01) |
| `7345-lb-01` | HAProxy load balancer |

## Files

| File | Description |
|------|-------------|
| `0-custom_http_response_header` | Installs nginx and adds an `X-Served-By` response header containing the server's hostname, so you can see which web server answered a request |
| `1-install_load_balancer` | Installs HAProxy on lb-01 and distributes traffic between web-01 and web-02 using the roundrobin algorithm |

## Usage

Run `0-custom_http_response_header` on web-01 and web-02, and `1-install_load_balancer` on lb-01.

To check that it works, query the load balancer several times:

```bash
for i in 1 2 3 4; do curl -sI LB_IP | grep X-Served-By; done
```

The header should alternate between `7345-web-01` and `7345-web-02`.
