FROM stilliard/pure-ftpd

RUN apt-get update && apt-get install -y \
    openssl

RUN getent group ftpgroup || groupadd -g 2001 ftpgroup
RUN getent passwd ftpuser || useradd -u 3001 -s /bin/false -d /home/ftpuser -m -c "Pure-FTPd user" -g ftpgroup ftpuser

# Définir un mot de passe fixe
RUN echo 'ftpuser:ftproot' | chpasswd

RUN mkdir -p /home/ftpuser/ftp && \
    chown -R ftpuser:ftpgroup /home/ftpuser/ftp

RUN echo "2" > /etc/pure-ftpd/conf/TLS
RUN mkdir -p /etc/pure-ftpd/passwd

# Ajouter l'utilisateur FTP au fichier de mots de passe
RUN mkdir -p /etc/pure-ftpd/passwd && \
    touch /etc/pure-ftpd/passwd/pureftpd.passwd && \
    chmod 600 /etc/pure-ftpd/passwd/pureftpd.passwd


EXPOSE 21 30000-30009

CMD /usr/sbin/pure-ftpd -c 50 -C 10 -l puredb:/etc/pure-ftpd/pureftpd.pdb -E -j -R -P $PUBLICHOST -p 30000:30009 -H -4 -b
