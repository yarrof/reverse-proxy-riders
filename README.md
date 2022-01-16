# Reverse Proxy Riders

## Story

You're participating in an online programming course and the instructors asked
your team to setup a reverse proxy and a few webservers behind it, each running on a different physical host.

Unfortunately, it turned out that only one member of the team has a public IPv4 address so ... running the webservers will be a challenge.

Luckily you've been warned about this and got hints about a solution in this pickle: SSH reverse tunnelling, also know as reverse port forwarding through SSH!



## What are you going to learn?

- What is a reverse proxy
- What is the different between a (forward) proxy compared to a reverse proxy
- Learning about `nginx` webserver
- Configuring `nginx` as a reverse proxy
- How to use SSH to create (forward) and reverse tunnels (aka port forwarding)

## Tasks

1. Install the `nginx` webserver and make sure it starts on a boot like a good service does.
    - Installed `nginx` on each team member's machine
    - `nginx` starts on boot on all team member's machine

2. Configure static webservers for every team member (other than the one who runs the proxy server) to serve a static `index.html` containing something specific to specific team members.
    - Created an `index.html` with some personal info for each team member
    - `nginx` is used to serve the `index.html` on port `8080`
    - `curl http;//localhost:8080/` or `curl http://localhost:8080/index.html` fetches the `index.html`'s content on each team member's machine


3. Team members running the webservers access the main machine via SSH and create reverse tunnels/port forwards to the remove system. The reverse proxy then can access the webservers via its loopback interface (`localhost` or `127.0.0.1`).
    - A Linux account with SSH access is created for other team members on the main machine (running the reverse proxy)
    - Created `ssh` reverse tunnels (with `ssh -R`) to the server running the reverse proxy
    - The local port is the webserver's port `8080`
    - The remote port is one of the following `8001`, `8002`, `8003`, `8004`, etc., each team member using a different one

4. Setup a _server_ in `nginx` with multiple _locations_ which proxy/forward request to servers (available via reverse SSH tunnels on `localhost`).
    - Configured the `nginx` reverse proxy server to listen on port `80`
    - Configured the proxy server to forward requests to tunnels listening on ports `8001`, `8002`, etc. on `localhost`

Request forwarding is based on the requests path, e.g.in case of requests arriving to
- `http://<public IP>/a` is forwarded to `http://localhost:8001`
- `http://<public IP>/b` is forwarded to `http://localhost:8002`
- `http://<public IP>/c` is forwarded to `http://localhost:8003`
- ...

5. Setup a (sub)domain name for the main server
    - An `A` DNS record is configured with the team member's public IP running the reverse proxy

6. Test if the reverse proxy is reachable via an IP address and/or a domain name and that it correctly forwards traffic to required server behind the scenes based on the request's path.
    - Sending a request to paths like `/a`, `/b`, `/c`, etc. using the _public IP_ of the reverse proxy
fetches the `index.html` of different team members, e.g.

- `curl http://<public IP>/a` fetches the first team members `index.html`
- `curl http://<public IP>/b` fetches the second team member's `index.html`
- `curl http://<public IP>/c` fetches the third team member's `index.html`
- ...
    - Sending a request to paths like `/a`, `/b`, `/c`, etc. using the _domain name_ of
the reverse proxy fetches the `index.html` of different team members, e.g.

- `curl http://<domain>/a` fetches the first team members `index.html`
- `curl http://<domain>/b` fetches the second team member's `index.html`
- `curl http://<domain>/c` fetches the third team member's `index.html`
- ...

## General requirements

- Only one team member (the one who has a public IP) runs a reverse proxy
- Others in the team run a webserver serving static content
- Configuration and other static files are save in the project's Git repository

## Hints

- When reading about [SSH Tunnel (SSH Port forwarding)](https://blog.leiwang.info/posts/ssh-tunnel/
) read only about _Local_ and _Remote Forwarding_
- When reading about [`nginx` in the _Beginners Guide_](http://nginx.org/en/docs/beginners_guide.html) you can skip the part about _FastCGI_
- `index.html` is the default _welcome_ file for most webservers; if you're serving a directory that contains a file named as such then it'll be automatically served to clients, e.g. `curl http://localhost:1234/` returns the contents of the `index.html` if the webserver behind the scenes is serving a root directory containing an `index.html`
- When fiddling with `nginx.conf` you should only use the following `nginx` configuration directives: `http`, `server`, `listen`, `location`, `root`, `proxy_pass` and `rewrite` (there are a lot of variants and extras one can set, but with these are enough)
- If `nginx` is barking at you because of a missing `events` directive just use an empty one: `events {}`
- The `rewrite` directive is needed to change a request's _path_ arriving at the reverse proxy before passing that onto another server

## Background materials

- [What are forward and reverse proxies?](https://www.cloudflare.com/learning/cdn/glossary/reverse-proxy/)
- [`nginx` Beginners Guide](http://nginx.org/en/docs/beginners_guide.html)
- [How to use `nginx` as a reverse proxy](https://www.techrepublic.com/article/how-to-use-nginx-as-a-reverse-proxy/)
- [SSH Tunnel (SSH Port forwarding)](https://blog.leiwang.info/posts/ssh-tunnel/)
- [Visual representation of SSH tunneling](https://unix.stackexchange.com/questions/115897)
- [SSH `-R` manual](https://man.openbsd.org/ssh#R)
