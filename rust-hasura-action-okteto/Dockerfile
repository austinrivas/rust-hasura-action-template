FROM openfaas/of-watchdog:0.7.7 as watchdog

FROM austinrivas/rust-actix-builder:stable

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog

ENV FUNC_HOME_DIR=/home/rust

RUN mkdir -p ${FUNC_HOME_DIR} \
  && cargo install watchexec \
  && chmod +x /usr/bin/fwatchdog \
  && addgroup --system --gid 12000 fn_group \ 
  && adduser --system --home ${FUNC_HOME_DIR} -u 12000 --gid 12000 fn_user \
  && chown fn_user:fn_group -R ${FUNC_HOME_DIR} \
  && chown fn_user:fn_group -R /usr/local \
  && chmod 777 /tmp \
  && echo 'Welcome to your development environment. Happy coding!' > /etc/mod \
  && echo 'export PS1="\[\e[32m\]okteto\[\e[m\]> "' >> .bashrc

USER fn_user

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

# OpenFaaS vars
ENV cgi_headers="true"
ENV exec_timeout="10s"
ENV write_timeout="15s"
ENV read_timeout="15s"

# Set up watchdog for HTTP mode
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"
ENV REPO_URL=https://raw.githubusercontent.com/austinrivas/rust-hasura-action-template/master
ENV TEMPLATE_SOURCE_URL=${REPO_URL}/template/rust-hasura-action
ENV WRAPPER_SOURCE_URL=${TEMPLATE_SOURCE_URL}/main
ENV HANDLER_SOURCE_URL=${TEMPLATE_SOURCE_URL}/function

ADD --chown=fn_user:fn_group ${WRAPPER_SOURCE_URL}/src/main.rs ./main/src/
ADD --chown=fn_user:fn_group ${WRAPPER_SOURCE_URL}/Cargo.toml ./main/
ADD --chown=fn_user:fn_group ${WRAPPER_SOURCE_URL}/Cargo.lock ./main/

ADD --chown=fn_user:fn_group ${HANDLER_SOURCE_URL}/src/lib.rs ./function/src/
ADD --chown=fn_user:fn_group ${HANDLER_SOURCE_URL}/Cargo.toml ./function/
ADD --chown=fn_user:fn_group ${HANDLER_SOURCE_URL}/Cargo.lock ./function/

RUN cargo build --manifest-path ./main/Cargo.toml \
    && cargo build --manifest-path ./function/Cargo.toml