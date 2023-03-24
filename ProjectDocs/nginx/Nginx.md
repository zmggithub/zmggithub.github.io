一、反向代理

	upstream  cluster {
		
		server 127.0.0.1:8848;
	
		server 127.0.0.1:8846;
	
		server 127.0.0.1:8844;
	
	}
	
	server {
	
		linsten 8847;
	
		server_name localhost;
	
		location /nacos/ {
	
			proxy_pass http://cluster/nacos/;
	
		}
	
	}
