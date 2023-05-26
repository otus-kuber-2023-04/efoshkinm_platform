FROM nginx:stable

RUN   mkdir /app \
      &&  usermod -u 1001 nginx \
      && rm /etc/nginx/conf.d/default.conf
COPY index.html homework.html /app 
COPY homework.conf /etc/nginx/conf.d
WORKDIR /app

EXPOSE 80

CMD ["-g", "daemon off;"]
ENTRYPOINT ["nginx"]
