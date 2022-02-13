FROM debian:11

RUN apt-get update    && \
      apt-get install -y \
        curl             \
        hey           && \
      rm -rf /var/lib/apt/lists/*

RUN echo 'hey -z 10m -c 100 -H "API-KEY: ingress-nginx" -H "API-SECRET-KEY: awesome" -host "localhost" http://127.0.0.1/hello' > ~/.bash_history

