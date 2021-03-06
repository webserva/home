
#### How to create a client cert

<b>Please be redirected to: https://github.com/webserva/webserva/blob/master/docs/register-cert.md</b>

You can use our `bash` cert creation script using your account name, as per https://telegram.me/WebServaBot

Our bot will propose a bash script rendered with your account name, as per the authoratitive Telegram.org username.

```shell
curl -s https://webserva.com/cert-script/ACCOUNT
```
where you should substitute `ACCOUNT` for your RedisHub account name i.e. Telegram.org username.

In general, we recommend reviewing any script first before executing it as follows:
```shell
curl -s https://webserva.com/cert-script/ACCOUNT | bash
```

To see the commands being executed including `openssl` you can use `bash -x` as follows:
```shell
curl -s https://webserva.com/cert-script/ACCOUNT | bash -x
```

Query paramaters include:
- `id` - the client cert id e.g. `admin`
- `role` - the client cert tole e.g. `admin`
- `archive` - archive `~/.webserva/live` to `~/webserva/archive/TIMESTAMP`

The content of this script is as follows when run with a placeholder `ACCOUNT` account name:
```shell
# Curl this script and pipe into bash as follows to create key dir ~/.redishub/live:
# curl -s 'https://secure.redishub.com/cert-script/ACCOUNT' | bash
#
(
  set -u -e
  account='ACCOUNT'
  role='admin'
  id='admin'
  CN='ws:ACCOUNT:admin:admin' # unique cert name (certPrefix, account, role, id)
  OU='admin' # role for this cert
  O='ACCOUNT' # account name
  dir=~/.webserva/live # must not exist, or be archived
  # Note that the following files are created in this dir:
  # account privkey.pem cert.pem privcert.pem privcert.p12 x509.txt cert.extract.txt
  commandKey='cert-script'
  serviceUrl='https://secure.redishub.com'
  archive=~/.webserva/archive
  certWebhook="${serviceUrl}/create-account-telegram/${account}"
  mkdir -p ~/.webserva # ensure default dir exists
  if [ -d ~/.webserva/live ]
  then
    echo "Directory ~/.webserva/live already exists. Try add '?archive' query to the URL."
  else
    mkdir ~/.webserva/live && cd $_ # error exit if dir exists
    curl -s https://raw.githubusercontent.com/webserva/webserva/master/bin/cert-script.sh -O
    cat cert-script.sh
    sha1sum cert-script.sh
    curl -s https://webserva.com/assets/cert-script.sh.sha1sum
    echo 'Press Ctrl-C in the next 8 seconds if the above hashes do not match'
    sleep 8
    source <(cat cert-script.sh)
  fi
)
```
where we fetch https://raw.githubusercontent.com/evanx/redishub/master/bin/cert-script.sh
```
  echo "${account}" > account
  if openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -subj "/CN=${CN}/OU=${OU}/O=${O}" \
    -keyout privkey.pem -out cert.pem
  then
    openssl x509 -text -in cert.pem > x509.txt
    grep 'CN=' x509.txt
    cat cert.pem | head -3 | tail -1 | tail -c-12 > cert.extract.pem
    cat privkey.pem cert.pem > privcert.pem
    openssl x509 -text -in privcert.pem | grep 'CN='
    curl -s -E privcert.pem "$certWebhook" ||
      echo "Registered account ${account} ERROR $?"
    if ! openssl pkcs12 -export -out privcert.p12 -inkey privkey.pem -in cert.pem
    then
      echo "ERROR $?: openssl pkcs12 ($PWD)"
      false # error code 1
    else
      echo "Exported $PWD/privcert.p12 OK"
      pwd; ls -l
      sleep 2
      curl -s https://redishub.com/cert-script-help/${account}
      curl -s https://raw.githubusercontent.com/webserva/webserva/master/docs/install.wscurl.txt
      certExtract=`cat cert.extract.pem`
      echo "Try https://telegram.me/WebServaBot '/grant $certExtract'"
    fi
  fi
```

### How to register a client cert

```shell
curl -E ~/.webserva/privcert.pem https://secure.webserva.com/register-cert
```


#### Troubleshooting

```shell
openssl x509 -text -in ~/.webserva/live/privcert.pem | grep 'CN='
```
