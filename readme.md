# devtun

A simpler(?) replacement for `ngrok`. This is how it works:

1. Creates a temporary `nginx` configuration file on the host server for the specified subdomain and port
2. Creates an SSH tunnel mapping the remote port to the corresponding local port
3. Deletes the temporary `nginx` configuration file when you quit

Though it's kind of hacky, it's easier for me to reason about and debug.

## Setup

This example uses `tun.publicscience.co` as the host.

### Let's Encrypt wildcard certificate

(Using `certbot`)

Setup your `nginx` conf, for example `/etc/nginx/conf.d/tun.publicscience.co.conf`:

```
server {
    server_name tun.publicscience.co;

    access_log /var/log/nginx/$host;

    location / {
            proxy_pass http://localhost:3000/;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_redirect off;
    }

    error_page 502 /50x.html;
    location = /50x.html {
            root /usr/share/nginx/html;
    }
}
```

Note that this forwards requests to `localhost:3000`.

Start the certificate process on the host:

```
sudo certbot certonly --manual -d tun.publicscience.co -d *.tun.publicscience.co --server https://acme-v02.api.letsencrypt.org/directory
```

Then follow the instructions. You will have to add a `TXT` record for `tun.publicscience.co`.

The other step requires a specific file, `.well-known/...` (full path will be provided in the instructions). It might be easiest to set up port forwarding to your local machine, such that requests to the host (which are passed to port 3000) are then forwarded to your local machine's port 8000:

```
ssh -N -R 3000:localhost:8000 tun.publicscience.co
```

Then create the `.well-known/...` file locally. In the parent directory of the `.well-known` directory, run a local python server: `python -m http.server 8000` to serve this directory. Now you can complete the `certbot` process.

## Usage

Once that's all setup, you can run:

```
./portal PORT SUBDOMAIN HOST
```

For example, if running a local Flask server on port 3000 that you want to expose at `foobar.tun.publicscience.co`, run:

```
./portal 3000 foobar tun.publicscience.co
```

It's helpful to think of it as connecting local port `3000` _to_ `foobar.tun.publicscience.co`.
