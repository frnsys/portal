# devtun

A simpler(?) replacement for `ngrok`

## Setup

This example uses `tun.publicscience.co` as the host.

### Let's Encrypt wildcard certificate

(Using `certbot`)

Setup your `nginx` conf:

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

Start the certificate process:

```
sudo certbot certonly --manual -d tun.publicscience.co -d *.tun.publicscience.co --server https://acme-v02.api.letsencrypt.org/directory
```

Then follow the instructions. You will have to add a `TXT` record for `tun.publicscience.co` as well as create a file at `.well-known/...` (full path will be provided in the instructions). Since this example is actually port forwarding, on the local machine run:

```
ssh -N -R 3000:localhost:8000 tun.publicscience.co
```

And then run `python -m http.server 8000` and create the `.well-known/...` file in the folder you run that local server in.

## Usage

Once that's all setup, you can run:

```
./expose.sh SUBDOMAIN PORT
```

For example, if running a local Flask server on port 3000 that you want to expose at `foobar.tun.publicscience.co`, run:

```
./expose.sh foobar 3000
```