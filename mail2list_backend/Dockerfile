FROM rust:latest as builder
#FROM ekidd/rust-musl-builder:stable as builder

RUN USER=root cargo new --bin docker_rust_web
WORKDIR ./docker_rust_web
RUN USER=root cargo new --bin mail2list_archive_unsubscribe \
    && USER=root cargo new --bin mail2list_common \
    && USER=root cargo new --bin mail2list_web
COPY ./Cargo.toml ./Cargo.toml
COPY ./mail2list_archive_unsubscribe/Cargo.toml ./mail2list_archive_unsubscribe/Cargo.toml
COPY ./mail2list_common/Cargo.toml ./mail2list_common/Cargo.toml
COPY ./mail2list_web/Cargo.toml ./mail2list_web/Cargo.toml

COPY ./debian/config  /usr/local/cargo
RUN CARGO_HTTP_MULTIPLEXING=false cargo fetch

RUN cargo build --release  \
    && rm src/*.rs mail2list_archive_unsubscribe/src/*.rs  mail2list_common/src/*.rs mail2list_web/src/*.rs target/release/deps/mail2list_archive_unsubscribe* target/release/deps/mail2list_common* target/release/deps/mail2list_web*

ADD . ./


RUN cargo build --release

FROM debian:bookworm-slim

ARG APP=/usr/src/app

RUN echo \
   deb http://mirrors.aliyun.com/debian/ buster main non-free contrib \
deb http://mirrors.aliyun.com/debian-security buster/updates main \
deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib\
deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib\
deb-src http://mirrors.aliyun.com/debian-security buster/updates main\
deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib\
deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib\
deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib\
> /etc/apt/sources.list

RUN apt-get clean\
  && apt-get update \
  && apt-get install -y ca-certificates tzdata \
  && apt-get install -y musl-tools openssl \
  && apt-get install -y iputils-ping \
  && apt-get install -y  telnet \
  && rm -rf /var/lib/apt/lists/*

EXPOSE 9999

ENV TZ=Etc/UTC \
  APP_USER=appuser

RUN groupadd $APP_USER \
  && useradd -g $APP_USER $APP_USER \
  && mkdir -p ${APP}

COPY --from=builder /docker_rust_web/target/release/mail2list_web ${APP}/mail2list_web
COPY --from=builder /docker_rust_web/target/release/mail2list_archive_unsubscribe ${APP}/mail2list_archive_unsubscribe
COPY --from=builder /docker_rust_web/db/mail_list.sql /usr/src/app/db/mail_list.sql
COPY --from=builder /docker_rust_web/debian/run.sh /usr/src/app/debian/run.sh

RUN chown -R $APP_USER:$APP_USER ${APP}

USER $APP_USER
WORKDIR ${APP}

RUN ["chmod", "+x","/usr/src/app/debian/run.sh"]
ENTRYPOINT ["/usr/src/app/debian/run.sh"]
