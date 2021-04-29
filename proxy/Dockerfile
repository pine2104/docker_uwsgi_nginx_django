FROM nginx:latest
LABEL maintainer="ychanc2104@gmail.com"


COPY nginx.conf /etc/nginx/nginx.conf
COPY sample.conf /etc/nginx/sites-available/sample.conf
COPY uwsgi_params /etc/nginx/uwsgi_params

RUN mkdir -p /etc/nginx/sites-enabled/ && \
    ln -s /etc/nginx/sites-available/sample.conf /etc/nginx/sites-enabled/sample.conf

CMD ["nginx", "-g", "daemon off;"]

