FROM debian:stretch-slim

LABEL maintainer="JoelKle <34544090+JoelKle@users.noreply.github.com>"

ENV container docker
ENV DEBIAN_FRONTEND noninteractive

# Enable systemd.
RUN apt-get update ; \
  apt-get install -y bash sudo python3 python3-apt systemd systemd-sysv ; \
  apt-get clean ; \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* ; \
  rm -rf /lib/systemd/system/multi-user.target.wants/* ; \
  rm -rf /etc/systemd/system/*.wants/* ; \
  rm -rf /lib/systemd/system/local-fs.target.wants/* ; \
  rm -rf /lib/systemd/system/sockets.target.wants/*udev* ; \
  rm -rf /lib/systemd/system/sockets.target.wants/*initctl* ; \
  rm -rf /lib/systemd/system/sysinit.target.wants/systemd-tmpfiles-setup* ; \
  rm -rf /lib/systemd/system/systemd-update-utmp*

VOLUME [ "/sys/fs/cgroup" ]
CMD ["/lib/systemd/systemd"]
