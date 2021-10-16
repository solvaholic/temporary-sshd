# didactic-octo-succotash

If you frequently SSH to an online service like GitHub, some firewalls and security gateways may block your connections. In this case you may see an error like:

```text
kex_exchange_identification: Connection closed by remote host
```

To determine whether it's the remote service blocking the connection, or something on your end, make a temporary SSH server available and check whether you can connect to it.

## Prerequisites

The instructions below assume you're already set up to use ngrok and Docker.

Use [Docker Desktop](https://www.docker.com/products/docker-desktop) in macOS and Windows. Follow [instructions on docker.com](https://docs.docker.com/engine/install/) to install Docker in Linux.

If you haven't created an [ngrok](https://ngrok.com) account yet, [go ahead and do that](https://dashboard.ngrok.com/signup).

## Instructions

Visit [your ngrok dashboard](https://dashboard.ngrok.com/get-started/your-authtoken) and copy your authtoken. Then start the temporary SSH server and make it available remotely:

```bash
_token="PasteYourTokenHere"

docker run --detach --tty --rm --name sshd linuxserver/openssh-server

docker run --interactive --tty --rm \
  --name ngrok --link sshd -e NGROK_AUTHTOKEN="$_token" \
  ngrok/ngrok tcp sshd:2222
```

ngrok will display tunnel details, for example:

```text
Region                        United States (us)                                                                         
Web Interface                 http://0.0.0.0:4040                                                                        
Forwarding                    tcp://999.tcp.ngrok.io:12345 -> localhost:2222                                               
```

Get the remote hostname and port number from the _Forwarding_ string and provide them to `ssh`:

```bash
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null \
  -p 12345 999.tcp.ngrok.io
```

Following these instructions, a successful connection to the remote service will look similar to this:

```text
username@999.tcp.ngrok.io: Permission denied (publickey,keyboard-interactive).
```

_Pro Tip:_ Those `-o` options prevent `ssh` from adding this temporary host to your `known_hosts` file. If you're going to use ngrok this way more than once or twice, consider adding those options to your `ssh_config`. For example:

```text
# See `man ssh_config` for details and examples.
Host *.tcp.ngrok.io
  User anonymous
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
```

Remember to clean up when you're done: Use CTRL+C to stop the ngrok container, and `docker stop sshd` to stop the OpenSSH server container.
