FROM nginx:alpine
#COPY FILE
COPY landing/ /var/www/html/
# Nginx config
RUN rm /etc/nginx/conf.d/default.conf
COPY landing/landing-page.conf /etc/nginx/conf.d/
