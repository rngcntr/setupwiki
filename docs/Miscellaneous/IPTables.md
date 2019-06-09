# iptables

!!! quote "Source: [stackoverflow.com](https://stackoverflow.com/questions/11164672/list-of-ip-space-used-by-facebook)"

Find the ip subnets of facebook:

``` console
$ whois -h whois.radb.net -- '-i origin AS32934' | grep ^route
```

Paste the subnets manually to iptables:

``` console
# iptables -A OUTPUT -d xxx.xxx.xxx.xxx/xx -j REJECT
```

Or paste them automatically with this script:
``` console
# for subnet in $(whois -h whois.radb.net -- '-i origin AS32934' | grep ^route: | sed 's/^route:\s*//g'); do
#   iptables -A OUTPUT -d $subnet -j REJECT
# done;
# for subnet in $(whois -h whois.radb.net -- '-i origin AS32934' | grep ^route6: | sed 's/^route6:\s*//g'); do
#   ip6tables -A OUTPUT -d $subnet -j REJECT
# done;
```

## Clearing iptables

!!! quote "Source: [howtoforge.com](https://www.howtoforge.com/blocking-facebook-web-trackers-at-the-firewall-for-extra-privacy)"

Simply run the following three commands:

``` console
# iptables -Z
# iptables -F
# iptables -X
```
