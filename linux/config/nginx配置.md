```
user www www;
worker_processes 8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
error_log /var/log/nginx_error.log crit;
pid /var/run/nginx.pid;
worker_rlimit_nofile 60000;
events
        {
                use epoll; #epoll 是Linux内核中的一种可扩展IO事件处理机制
                worker_connections 8192;
                multi_accept on;
        }
http {
                include mime.types;
                default_type application/octet-stream;
                server_names_hash_bucket_size 128;
                server_names_hash_max_size 512;
                client_header_buffer_size 32k;
                client_header_timeout 10;
                client_body_buffer_size 16k;
                client_body_timeout 10;
                large_client_header_buffers 16 8k;
                client_max_body_size 100m;
                directio 4m;
                server_tokens off;
                sendfile on;
                send_timeout 2;
                tcp_nopush on;
                tcp_nodelay on;
                types_hash_max_size 2048;
                keepalive_timeout 30;
                keepalive_requests 100000;
                reset_timedout_connection on;
                open_file_cache max=200000 inactive=20s;
                open_file_cache_valid 30s;
                open_file_cache_min_uses 2;
                open_file_cache_errors on;
                gzip on;
                gzip_min_length 1k;
                gzip_disable "MSIE [1-6]\.";
                gzip_buffers 16 8k;
                gzip_http_version 1.1;
                gzip_comp_level 5;
                gzip_proxied expired no-cache no-store private auth;
                gzip_types image/gif image/jpeg text/plain application/x-javascript text/css application/xml text/javascript;
                gzip_vary on;
                tfs_body_buffer_size 2m;
                tfs_send_timeout 3s;
                tfs_connect_timeout 3s;
                tfs_read_timeout 3s;


        tfs_upstream tfs_ns {
                server tfs_server_ip:8100;
                type ns;
        }

    server {
          listen 80;
          server_name my_ip;

          tfs_keepalive max_cached=100 bucket_count=10;
          tfs_log /opt/nginx/logs/cronolog/tfs_access.log;

          location / {
              tfs_pass tfs://tfs_ns;
          }
    }
}

nginx 多域名反向代理
server {
        listen       80;
        server_name  sunc.com;

        #charset koi8-r;

        #access_log  /opt/nginx/logs/host.access.log  main;

        location / {
           proxy_pass http://192.168.1.88:3003;
           # root   html;
           # index  index.html index.htm;
        }

        error_page  404              /404.html;
        location = /404.html{
            root   html;
        }

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

    }
    server {
        listen  80;
        server_name sun.com;
        location / {
             proxy_pass http://127.0.0.1:8080;
        }
    }
```