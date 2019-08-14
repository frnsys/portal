#!/bin/bash
set -e

export HOST=tun.publicscience.co
export SUBDOMAIN=$1
export PORT=$2

if [[ -z "${SUBDOMAIN}" ]] || [[ -z "${PORT}" ]]; then
    echo "Define SUBDOMAIN and PORT"
    exit 1
fi

CONF=$(cat nginx.conf.template | envsubst '$SUBDOMAIN $PORT $HOST')
ssh -t $HOST 'echo "'"$CONF"'" | sudo tee /etc/nginx/sites-enabled/'$SUBDOMAIN'.'$HOST'; sudo service nginx restart'

function cleanup {
    ssh -t $HOST 'sudo rm /etc/nginx/sites-enabled/'$SUBDOMAIN'.'$HOST'; sudo service nginx restart'
}
trap cleanup EXIT

ssh -N -R $PORT:localhost:$PORT $HOST