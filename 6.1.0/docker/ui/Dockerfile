FROM reg.chehejia.com/chj-k8s-change/public/skywalking/base:6.1.0

# 时区修改为东八区
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

ENV JAVA_OPTS=" -Xms256M " \
    SW_OAP_ADDRESS="127.0.0.1:12800" \
    SW_TIMEOUT="20000"

WORKDIR skywalking

RUN set -eux; \
    rm -rf "config"; rm -rf "oap-libs"; \
    rm -rf "agent";

COPY docker-entrypoint.sh .
COPY logback.xml webapp/

EXPOSE 8080

ENTRYPOINT ["bash", "docker-entrypoint.sh"]