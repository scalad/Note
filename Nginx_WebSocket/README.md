### Nginx反向代理WebSocket ###

	worker_processes  1;
	events {
	    worker_connections  1024;
	}
	
	http {
	    include       mime.types;
	    default_type  application/octet-stream;
	    sendfile        on;
	    autoindex       on;
	
	    #keepalive_timeout  0;
	    keepalive_timeout  65;
	
	    map $http_upgrade $connection_upgrade {
	        default upgrade;
	        '' close;
	    }
	
	    server {
	        listen       80;
	        server_name  example.com;
	
	        #charset koi8-r;
	
	        #access_log  logs/host.access.log  main;
	
	        location / {
	            proxy_pass http://120.27.114.229/;
		    proxy_redirect off;
	
		    proxy_set_header X-Real-IP $remote_addr;
		    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		    proxy_set_header Host $http_host;
		    proxy_set_header X-NginX-Proxy true;
	
		    proxy_http_version 1.1;
	    	    proxy_set_header Upgrade $http_upgrade;
		    proxy_set_header Connection "upgrade";
	        }
	    }
	}
	
