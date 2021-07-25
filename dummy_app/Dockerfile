FROM busybox:stable

COPY index.html /www/

CMD ["httpd", "-f", "-p", "80", "-h", "/www"]
