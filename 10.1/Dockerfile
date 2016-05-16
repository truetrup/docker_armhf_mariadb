FROM armbuild/debian:latest

ENV MARIADB_MAJOR 10.1
ENV MARIADB_VERSION 10.1.14+maria-1~jessie

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys F447D848D4FDED4A  \
	&& echo "deb http://apt.drugc0de.tk/ jessie main" >> /etc/apt/sources.list.d/custom.list \
	&& apt-get update
	
RUN mkdir /docker-entrypoint-initdb.d

RUN apt-get install --no-install-recommends --no-install-suggests -y --force-yes \
	# install "pwgen" for randomizing passwords
	pwgen \
	socat


RUN { \
		echo mariadb-server-$MARIADB_MAJOR mysql-server/root_password password 'unused'; \
		echo mariadb-server-$MARIADB_MAJOR mysql-server/root_password_again password 'unused'; \
	} | debconf-set-selections \
	&& apt-get update \
	&& apt-get install -y \
		mariadb-server=$MARIADB_VERSION \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -rf /var/lib/mysql \
	&& mkdir /var/lib/mysql

# comment out a few problematic configuration values
# don't reverse lookup hostnames, they are usually another container
RUN sed -Ei 's/^(bind-address|log)/#&/' /etc/mysql/my.cnf \
	&& echo 'skip-host-cache\nskip-name-resolve' | awk '{ print } $1 == "[mysqld]" && c == 0 { c = 1; system("cat") }' /etc/mysql/my.cnf > /tmp/my.cnf \
	&& mv /tmp/my.cnf /etc/mysql/my.cnf

VOLUME /var/lib/mysql

COPY docker-entrypoint.sh /

ENTRYPOINT ["/docker-entrypoint.sh"]

EXPOSE 3306
CMD ["mysqld"]
