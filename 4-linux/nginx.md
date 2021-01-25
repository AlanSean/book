
server {
    listen 80;
    server_name book.hqboke.cn;

    location /{
        root /home/book/;
        index  index.html index.htm;
    }
      
    error_page 404  500 502 503 504 /50x.html;
    return 301 https://$server_name$request_uri;
}

server {
    listen	 443 ssl;
    server_name book.hqboke.cn  hqboke.cn; #填写绑定证书的域名
    server_name_in_redirect off;
    ssl_certificate         ssl/hqboke/1_book.hqboke.cn_bundle.crt;
    ssl_certificate_key     ssl/hqboke/2_book.hqboke.cn.key;

    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #按照这个协议配置
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;#按照这个套件配置
    ssl_prefer_server_ciphers on;
    root /home/book/;
    location ~*\.(js|css|png|jpg|jpeg|gif|ico)$ {
     	expires 30d;
	add_header Pragma public;
        add_header Cache-Control "public";
    }
    location /{
        root /home/book/;
        index index.php index.html index.htm;
	proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Connection       close;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_max_temp_file_size 0;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;

    }
    
}
