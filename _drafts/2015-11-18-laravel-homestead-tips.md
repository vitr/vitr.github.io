---
published: false
---

## Laravel's Homestead Tips

/etc/nginx/nginx.conf
in http { section add
	large_client_header_buffers 4 160k;

