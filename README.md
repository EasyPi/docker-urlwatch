urlwatch
========

[![](https://github.com/easypi/docker-urlwatch/actions/workflows/build.yaml/badge.svg)](https://github.com/EasyPi/docker-urlwatch)

[![](http://dockeri.co/image/easypi/urlwatch)](https://hub.docker.com/r/easypi/urlwatch)

[urlwatch][1] is a tool for monitoring webpages for updates.

## docker-compose.yml

```yaml
version: "3.8"
services:
  urlwatch:
    image: easypi/urlwatch
    volumes:
      - ./data:/root/.urlwatch
    environment:
      - EDITOR=/usr/bin/vi
    restart: unless-stopped
```

## urls.yaml

```yaml
---

name: urlwatch
url: https://github.com/thp/urlwatch/tags
user_visible_url: https://github.com/thp/urlwatch
filter:
- xpath: '(//h4[@data-test-selector="tag-title"]/a)[1]'
- html2text: re
- strip:

---

name: shadowsocks-libev
url: https://api.github.com/repos/shadowsocks/shadowsocks-libev/releases/latest
user_visible_url: https://github.com/shadowsocks/shadowsocks-libev
filter:
- shellpipe: 'jq -r .tag_name'
- strip:

---

name: easypi
url: https://www.nslookup.io/api/v1/records
user_visible_url: https://www.nslookup.io/domains/easypi.duckdns.org/dns-records/#authoritative
method: POST
headers:
  Content-Type: application/json
data: '{"domain":"easypi.duckdns.org","dnsServer":"authoritative"}'
filter:
- jq:
    query: '.records.a.response.answer[0].ipInfo.query'
- re.sub:
    pattern: '"'
    repl: ''

...
```

## up and running

```bash
$ docker-compose up -d
$ docker-compose exec urlwatch sh
>>> urlwatch --test-reporter slack
Successfully sent message to Slack
>>> urlwatch --list
1: https://github.com/thp/urlwatch/releases/latest
2: https://github.com/shadowsocks/shadowsocks-libev/releases/latest
3: https://www.nslookup.io/domains/easypi.duckdns.org/dns-records/#authoritative
>>> urlwatch --test-filter 2
v3.3.5
>>> exit
```

## customizing cron schedule

### Create a crontab file

```
*/30 * * * * cd /root/.urlwatch && urlwatch --urls urls.yaml --config urlwatch.yaml --hooks hooks.py --cache cache.db
*/15 * * * * cd /root/.urlwatch && urlwatch --urls urls-every-15m.yaml --config urlwatch.yaml --hooks hooks.py --cache cache.db
```

See the [crontab manpage for details on format](https://man7.org/linux/man-pages/man5/crontab.5.html#DESCRIPTION).

### Mount the crontab file as a docker volume

```yaml
version: "3.8"
services:
  urlwatch:
    image: easypi/urlwatch
    volumes:
      - ./data:/root/.urlwatch
      - ./data/crontab:/etc/crontabs/root
    restart: unless-stopped
```

[1]: https://github.com/thp/urlwatch
