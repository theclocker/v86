
FROM i386/alpine:3.18.6

ENV KERNEL=lts
ENV HOSTNAME=localhost
ENV ROOT_PASSWORD=root
ENV PYTHONUNBUFFERED=1

# Installing base packages, you can add additional packages here (community repo is enabled)
RUN apk add openrc alpine-base agetty alpine-conf

# Install newer mkinitfs from edge (todo: remove this when 3.19+ has worked properly with 9pfs)
RUN apk add mkinitfs --no-cache --allow-untrusted --repository https://dl-cdn.alpinelinux.org/alpine/edge/main/ 

# Installing kernel and remove unused drivers for linux-lts
RUN if [ "$KERNEL" == "lts" ]; then \
    apk add linux-lts linux-firmware-none; \
else \
    apk add linux-$KERNEL; \
fi

# Setting root password and hostname
RUN echo "root:$ROOT_PASSWORD" | chpasswd
RUN setup-hostname $HOSTNAME

# Enable autologin into root on tty1, also you can use autologin.c instead agetty
# (see https://wiki.alpinelinux.org/wiki/TTY_Autologin#By_compiling_your_own_autologin_wrapper)
RUN sed -i 's/getty 38400 tty1/agetty --autologin root tty1 linux/' /etc/inittab

# Adding networking.sh script (works only on lts kernel yet)
# (https://github.com/copy/v86/blob/master/tools/docker/debian/networking.sh)
RUN if [ "$KERNEL" == "lts" ]; then \ 
    echo -e "rmmod ne2k-pci && modprobe ne2k-pci\nhwclock -s\nsetup-interfaces -a -r" > /root/networking.sh && \ 
    chmod +x /root/networking.sh; \ 
fi

# Enable minimal needed OpenRC services
# (see https://wiki.alpinelinux.org/wiki/Alpine_Linux_in_a_chroot#Preparing_init_services)
RUN for i in devfs dmesg mdev hwdrivers; do rc-update add $i sysinit; done
RUN for i in hwclock modules sysctl hostname syslog bootmisc; do rc-update add $i boot; done
RUN rc-update add killprocs shutdown

# Generate initramfs with 9p modules
RUN mkinitfs -F "ata base ide scsi virtio ext4 9p" $(cat /usr/share/kernel/$KERNEL/kernel.release)


# Enable serial console with autologin for ttyS0
RUN echo "ttyS0::respawn:/sbin/agetty --autologin root --keep-baud 115200,38400,9600 ttyS0 vt100" >> /etc/inittab

RUN echo 'echo "Serial console initialized"' > /etc/local.d/serial.start && \
    chmod +x /etc/local.d/serial.start && \
    ln -s /etc/init.d/local /etc/runlevels/default/


# Install python3 and jupyter
RUN apk add --update --no-cache python3 && ln -sf python3 /usr/bin/python
RUN python3 -m ensurepip
RUN pip3 install --no-cache --upgrade pip setuptools

RUN apk add gcc g++ musl-dev make python3-dev linux-headers


WORKDIR /app

COPY ./src /app

RUN pip install -r requirements.txt

