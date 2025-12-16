Домашнее задание к занятию 2 «Кластеризация и балансировка нагрузки» - Дорожкин Артем

Задание 1

Запустите два simple python сервера на своей виртуальной машине на разных портах
Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке
Настройте балансировку Round-robin на 4 уровне.
На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.
Решение 1
haproxy.cfg

    global
        log /dev/log    local0                                                                       
        log /dev/log    local1 notice
        chroot /var/lib/haproxy                                                                      
        stats socket /run/haproxy/admin.sock mode 660 level admin
        stats timeout 30s                                                                            
        user haproxy
        group haproxy                                                                                
        daemon
        
       # Default SSL material locations
        ca-base /etc/ssl/certs                                                                       
        crt-base /etc/ssl/private
                                                                                                     
        # See: https://ssl-config.mozilla.org/
        #server=haproxy&server-version=2.0.3&config=intermedia>
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECD>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POL>
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets                                  

    defaults                                                                                             
        log     global
        mode    http                                                                                 
        #option httplog
        option  dontlognull
        timeout connect 5s
        timeout client  50s
        timeout server  50s
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

    listen stats  #
        bind *:888
        mode http
        option httplog
        stats enable                                                                                 
        stats uri /stats
        stats refresh 5s                                                                              
        stats realm Haproxy\ Statistics
        
    frontend example                                                                        
        mode http
        bind :8088                                                                                    
        option httplog
        default_backend web_servers                                                                   
        #acl ACL_example.com hdr(host) -i example.com
        #use_backend web_servers if ACL_example.com                                                   

    backend web_servers    
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:7777 check
        server s2 127.0.0.1:7788 check

    listen web_tcp
        mode tcp
        bind *:1352
        balance roundrobin
        option tcplog
        server s1 127.0.0.1:7777 check inter 3s
        server s2 127.0.0.1:7788 check inter 3s


Cкриншот, где видно перенаправление запросов на разные серверы при обращении к HAProxy:


<img width="728" height="230" alt="image" src="https://github.com/user-attachments/assets/cac172fb-76ee-476e-81c6-e706a32875e5" />





Задание 2
Запустите три simple python сервера на своей виртуальной машине на разных портах
Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.


Решение 2


