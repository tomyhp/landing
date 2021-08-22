FROM nginx:alpine
#COPY FILE
COPY landingpage/ /var/www/html/
# Nginx config
RUN rm /etc/nginx/conf.d/default.conf
COPY landingpage/ops/landing-page.conf /etc/nginx/conf.d/
