FROM centos:7

MAINTAINER RJ Bergeron <rbergero@gmail.com>

# install packages
RUN yum -y install hostname parted lvm2 cryptsetup mdadm xfsprogs \
                   grub grub2-tools grub2 grub2-efi grub2-efi-modules \
                   shim efibootmgr \
                   augeas \
                   selinux-policy-targeted \
                   NetworkManager openssh-server openssh-clients cloud-init \
                   sudo extendroot passwd authconfig epel-release chrony \
                   systemd-networkd \
                   smartmontools ethtool lm-sensors \
                   vim-minimal \
                   tboot \
                   kernel \
                   && \
    yum clean all && rm -rf /var/cache/yum &&\
    find / -type d \( -name proc -o -name sys \) -prune -o -iname \*.rpmnew -exec rm {} \;

# add grub bits
ADD default-grub /etc/default/grub

# add user
RUN groupadd -g 1024 ejusdem && \
    useradd --uid 1024 --gid 1024 ejusdem && \
    usermod -G wheel ejusdem

# reconfigure sudo
RUN bash -c 'autosudo=$(mktemp) && \
    printf \#\!/bin/bash\\nsed\ -i\ -e\ \"s@^%%wheel.*@%%wheel\ ALL=\(ALL:ALL\)\ NOPASSWD:\ ALL@\"\ \${2} > "${autosudo}" && \
    chmod +x "${autosudo}" && \
    env EDITOR="${autosudo}" visudo && \
    rm "${autosudo}"'

# reconfigure cloud-init
RUN sed -i -e '/ - locale/d' /etc/cloud/cloud.cfg
ADD /cloud.cfg.d /etc/cloud/cloud.cfg.d

# systemd-networkd for network config please
RUN ln -sf /lib/systemd/system/systemd-networkd.service "/etc/systemd/system/multi-user.target.wants/systemd-networkd.service" && \
    mkdir -p "/etc/systemd/system/sockets.target.wants" && \
    ln -sf /lib/systemd/system/systemd-networkd.socket  "/etc/systemd/system/sockets.target.wants/systemd-networkd.service" && \
    ln -sf /dev/null "/etc/systemd/system/dhcpcd.service" && \
    ln -sf /dev/null "/etc/systemd/system/NetworkManager.service" && \
    ln -sf /dev/null "/etc/systemd/system/networking.service" && \
    ln -sf /dev/null "/etc/systemd/system/NetworkManager-wait-online.service" && \
    rm -f "/etc/systemd/system/dbus-org.freedesktop.NetworkManager.service" && \
    rm -f "/etc/systemd/system/multi-user.target.wants/NetworkManager.service" && \
    rm -f "/etc/systemd/system/dbus-org.freedesktop.nm-dispatcher.service" && \
    mkdir -p "/etc/systemd/network"
ADD /systemd-network /etc/systemd/network
RUN find /etc/systemd/network -type f -exec chmod a+r {} \;

# custom script to dump platform info to a directory for systemd hooks
ADD platform-info/platform-info.service /etc/systemd/system/platform-info.service
ADD platform-info/platform-info.sh      /usr/local/sbin/platform-info.sh
RUN ln -sf /etc/systemd/system/platform-info.service "/etc/systemd/system/multi-user.target.wants/platform-info.service"

# disable smartd on virtual hosts
ADD smartd.service.d /etc/systemd/system/smartd.service.d

# service to start console if we find a hvc0 console *shrug*
ADD hvc0-console.service /etc/systemd/system/hvc0-console.service

# image installation scripts
ADD /scripts /scripts
