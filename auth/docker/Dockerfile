FROM openresty/openresty:1.19.9.1-5-alpine-fat

# specific env vars must be enabled explicitly in the main section
RUN echo ""                >> /usr/local/openresty/nginx/conf/nginx.conf && \
    echo "env SECRET_KEY;" >> /usr/local/openresty/nginx/conf/nginx.conf && \
    luarocks install lua-resty-string

COPY auth.conf /etc/nginx/conf.d/auth.conf
