FROM tsjsdbd/slurmbase
MAINTAINER t00402375, From: https://github.com/GRomR1/docker-slurmbase

ADD scripts/start.sh /root/start.sh
RUN chmod +x /root/start.sh

ADD etc/supervisord.d/slurmctld.conf /etc/supervisor/conf.d/slurmctld.conf

CMD ["/bin/bash","/root/start.sh"]
