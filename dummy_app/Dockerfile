FROM busybox:stable

COPY index.html /www/

EXPOSE 80

CMD ["httpd", "-f", "-p", "80", "-h", "/www"]
