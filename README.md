Fail2ban filter for AbuseIPDB
=============================

This script is a refactoring and extension of the script written by
[Fanjoe and published in this blog entry](https://www.fanjoe.be/?p=3185).

It takes the current incoming connections to the host where the script is run,
checks those IPs against the [AbuseIP Database](https://www.abuseipdb.com/>) 
and blocks them in [Fail2Ban](https://www.fail2ban.org).

It allows you to set

* A whitelist of IPs which you don't want to be checked. Defaults to
  IPv4 and IPv6 localhost.
* A "Confidence" threshold above which the IP will be banned in fail2ban.
  It is compared against the newest report returned
  by AbuseIPDB. Defaults to 40%. 
* A list of ports to check in the incoming connections, egrep formatted.
  Defaults to port 22 SSH.
* A "not reported" cache file name to avoid querying AbuseIPDB too frequently.
  Defaults to /tmp/banabuseip_not_reported_cache.
* A "cache threshold", in seconds, during which the cached IP in the "not
  reported" file won't be checked again against AbuseIPDB.
  Defaults to 4 hs.
* Curl timeout, defaults to 6 seconds.
* Iptables TARGET used to ban IPs, to check if the IP is already blocked.
  Defaults to REJECT.

``Prerequisites``
-----------------

This script requires the following packages/binaries installed in your
system:

* curl (to query AbuseIPDB's API)
* iptables (to check if already banned)
* fail2ban (obviously)
* jq (to parse AbuseIPDB's responses)

``Installing and using the script``
-----------------------------------

1. Download and install the script somewhere in your $PATH and make it executable,
   ie. `/usr/local/bin/fail2ban_abuseipdb_filter`.

2. Modify it to suit your needs, particularly, the `abuseipdb_api_key` variable.

3. Add the fail2ban jail
```
cat >> /etc/fail2ban/jail.local << EOF
[banabuseip]
enabled = true
filter = banabuseip
logpath = /var/log/banabuseip.log
port = 22
EOF
```

4. Add the fail2ban filter
```
cat >> /etc/fail2ban/filter.d/banabuseip.local << EOF
[Definition]
failregex =
ignoreregex =
EOF
```

5. Add a cron entry to run this script regularly
```
crontab -e
```
```
*/3 * * * * /usr/local/bin/fail2ban_abuseipdb_filter >> /var/log/banabuseip.log
```

And that's it. You can run it manually, if you want.

``TODO``
--------

Lot of room for improvement! Any PR/suggestion is highly welcomed! :smile:
