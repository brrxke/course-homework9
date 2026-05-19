# course-homework9

### Task 1

PNG-файл находится в папке assets

### Task 2

```bash
docker run --rm --network nginx-proxy-docker_default alpine/curl -Ivsk https://nginx-proxy
Unable to find image 'alpine/curl:latest' locally
latest: Pulling from alpine/curl
f3f198be84dd: Pull complete
d70add8dfd13: Pull complete
Digest: sha256:16ed43d4038dd824e3b935d16d6763cac68aca3e7c5b4248f38409a85a13547c
Status: Downloaded newer image for alpine/curl:latest
* Host nginx-proxy:443 was resolved.
* IPv6: (none)
* IPv4: 172.19.0.3
*   Trying 172.19.0.3:443...
* ALPN: curl offers h2,http/1.1
} [5 bytes data]
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
} [1557 bytes data]
* SSL Trust: peer verification disabled
{ [5 bytes data]
* TLSv1.3 (IN), TLS handshake, Server hello (2):
{ [1210 bytes data]
* TLSv1.3 (IN), TLS change cipher, Change cipher spec (1):
{ [1 bytes data]
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
{ [25 bytes data]
* TLSv1.3 (IN), TLS handshake, Certificate (11):
{ [970 bytes data]
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
{ [264 bytes data]
* TLSv1.3 (IN), TLS handshake, Finished (20):
{ [52 bytes data]
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
} [1 bytes data]
* TLSv1.3 (OUT), TLS handshake, Finished (20):
} [52 bytes data]
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384 / X25519MLKEM768 / RSASSA-PSS
* ALPN: server accepted http/1.1
* Server certificate:
*   subject: C=RU; ST=State; L=City; O=Organization; OU=Department; CN=localhost
*   start date: May 19 15:55:04 2026 GMT
*   expire date: May 19 15:55:04 2027 GMT
*   issuer: C=RU; ST=State; L=City; O=Organization; OU=Department; CN=localhost
*   Certificate level 0: Public key type RSA (2048/112 Bits/secBits), signed using sha256WithRSAEncryption
* OpenSSL verify result: 12
*  SSL certificate verification failed, continuing anyway!
* Established connection to nginx-proxy (172.19.0.3 port 443) from 172.19.0.4 port 47460
* using HTTP/1.x
} [5 bytes data]
> HEAD / HTTP/1.1
> Host: nginx-proxy
> User-Agent: curl/8.19.0
> Accept: */*
>
* Request completely sent off
{ [5 bytes data]
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
{ [281 bytes data]
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
{ [281 bytes data]
< HTTP/1.1 200 OK
< Server: nginx/1.31.0
< Date: Tue, 19 May 2026 16:02:36 GMT
< Content-Type: text/plain
< Content-Length: 141
< Connection: keep-alive
< Expires: Tue, 19 May 2026 16:02:35 GMT
< Cache-Control: no-cache
< Strict-Transport-Security: max-age=31536000; includeSubDomains
<
HTTP/1.1 200 OK
* Connection #0 to host nginx-proxy:443 left intact
Server: nginx/1.31.0
Date: Tue, 19 May 2026 16:02:36 GMT
Content-Type: text/plain
Content-Length: 141
Connection: keep-alive
Expires: Tue, 19 May 2026 16:02:35 GMT
Cache-Control: no-cache
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

---

### Task 4

```bash
#!/bin/bash

export LANG=C

log_alert() {
    logger -t "SECURITY_MONITOR" -p auth.warning "$1"
}

check_and_fix() {
    local file="$1"
    local safe_mode="$2"

    if [ ! -e "$file" ]; then
        return
    fi

    local current_mode=$(stat -c "%a" "$file")

    local user_p=${current_mode: -3:1}
    local group_p=${current_mode: -2:1}
    local other_p=${current_mode: -1:1}

    local safe_user_p=${safe_mode: -3:1}
    local safe_group_p=${safe_mode: -2:1}
    local safe_other_p=${safe_mode: -1:1}

    if [ $(( (group_p & ~safe_group_p) | (other_p & ~safe_other_p) | (user_p & ~safe_user_p) )) -gt 0 ]; then
        log_alert "Insecure permissions on $file: $current_mode. Restoring to $safe_mode."
        chmod "$safe_mode" "$file"
        chown root:root "$file"
    fi
}

check_and_fix "/etc/passwd" "644"
check_and_fix "/etc/shadow" "600"
check_and_fix "/etc/sudoers" "440"
```

добавляем скрипт в крон

```bash
sudo crontab -e
```

```
0 * * * * /usr/local/bin/check_critical_perms.sh
```

Ухудшил права на /etc/passwd

```bash
sudo chmod 777 /etc/passwd
```

```ls -l /etc/passwd
-rwxrwxrwx 1 root root 3327 May 15 08:31 /etc/passwd
```

```bash
sudo /usr/local/bin/check_critical_perms.sh
```

```bash
 ls -l /etc/passwd
-rw-r--r-- 1 root root 3327 May 15 08:31 /etc/passwd
```

```bash
sudo journalctl -t SECURITY_MONITOR

May 19 13:18:19 kali SECURITY_MONITOR[11516]: Insecure permissions on /etc/passwd: 777. Restoring to 644.
```
