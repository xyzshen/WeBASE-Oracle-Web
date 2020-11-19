FROM nginx:stable-alpine as prod


# copy server config file for nginx
COPY oracle-web.conf /etc/nginx/conf.d

# copy files of dist into html directory
COPY index.html /usr/share/nginx/html
COPY static /usr/share/nginx/html

ENV NODE_ENV "production"

