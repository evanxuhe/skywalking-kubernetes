FROM reg.chehejia.com/chj-k8s-change/public/skywalking/base:6.1.0

# 时区修改为东八区
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV JAVA_OPTS=" -Xms256M " \
    SW_CLUSTER="standalone" \
    SW_STORAGE="h2"

WORKDIR skywalking

RUN set -eux; \
    rm -rf "config/log4j2.xml"; \
    rm -rf "webapp"; rm -rf "agent";

COPY log4j2.xml config/
COPY docker-entrypoint.sh .

EXPOSE 12800 11800 1234

ENTRYPOINT ["bash", "docker-entrypoint.sh"]
