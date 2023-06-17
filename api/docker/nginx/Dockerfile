FROM nginx:1.24

ARG UID
EXPOSE $UID

RUN adduser -u ${UID} --disabled-password --gecos "" appuser

COPY nginx.conf /etc/nginx/conf.d/