FROM alpine:3.14

# install nginx and nginx echo module
RUN apk add -u --no-cache nginx nginx-mod-http-echo

RUN mkdir -p /run/nginx

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf


EXPOSE 80

CMD ["nginx"]
