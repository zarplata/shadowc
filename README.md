# shadowc

**shadowc** is a client for [the secure login distribution
service](https://github.com/reconquest/shadowd).

**shadowc** is not a user management tool, shadowc is hash entries manager,
which communicate with **shadowd** server through REST API and request hash
entries for users, afterwards **shadowc** overwrite `/etc/shadow` file with new
hash entries.

## Usage

It's considered that the **shadowd** server is configured earlier and you have
SSL certificate and trusted **shadowd** hosts. If not, [see documentation
here](https://github.com/reconquest/shadowd).

After generating SSL certificate, you should copy `cert.pem` file to client
host into `/etc/shadowc/` directory. But be careful and do not copy private key
`key.pem`, this file should not leave **shadowd** hosts.

**shadowc** can be used either on initial server configuration or for changing
hash entries anytime when you need change passwords.

For running **shadowc** you can specify users which needs update of shadow
entry via `-u <user>` argument, it is possible to specify more than one user.

If you want to request all users from the pool, you can use flag `--all`. Then,
hash entries will be written for each user from the pool which is specified via
`-p`.

Flag `--update` can be used to request new hash entries for users with
non-empty hash in /etc/shadow file. Users without password access or which are
not known to shadowd service will not be updated.

Also you must specify **shadowd** addresses via `-s <addr>` argument. You can
specify more than one server, but all specified addresses should be trusted by
SSL certificate.

If hash tables was generated by tokens with pools you should specify pool name
via `-p <pool>` argument, and **shadowc** will request hashes for users with
this pool.

**shadowc** can create users when needed. Flag **-C** should be added in that
case, which will instruct **shadowc** to create user if it does not exist. By
default, **shadowc** will create user via invocation of `useradd -m
<username>`, but flags for `useradd` can be changed using `-g`. For example,
sudo-user with home dir can be created by passing flag `-g "-m -Gwheel".

### Additional Options
- `-c <cert>` — set specified certificate file path. (default:
  `/etc/shadowc/cert.pem`)
- `-f <file>` — set specified shadow file path. Can be usable if you use
  `chroot` on your server and shadowc runned outside the `chroot`. (default:
  `/etc/shadow`)

#### Example

Assume that, you have certificate file and two shadowd servers on
`shadowd0.in.example.com:8080` and `shadowd1.in.example.com:8080`, certificate
file has been copied to `/data/shadowc/cert.pem` on target server.

If you want to grant password access for users `john` and `smith` on some
server, you should invoke following command, which will obtain unique hashes
for that server for specified users and place them into `/etc/shadow` file.

After that, `john` and `smith` will be able to connect to the server using
passwords, which were set on **shadowd** server while generating hash tables
for that users.

```
shadowc -s shadowd0.in.example.com:8888 -s shadowd1.in.example.com:8080 \
    -u john -u smith \
    -c /data/shadowc/cert.pem
```

Afterwards shadowc will overwrite the shadow file (`/etc/shadow`) and change
hash entries for John and Smith.
