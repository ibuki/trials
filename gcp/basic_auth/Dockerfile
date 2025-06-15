FROM nginx:alpine
COPY default.conf /etc/nginx/conf.d/default.conf
COPY html/ /usr/share/nginx/html/
COPY auth/.htpasswd /etc/nginx/.htpasswd
EXPOSE 8080
