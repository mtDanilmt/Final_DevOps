# Dockerfile для базы данных с SSH
FROM postgres:15

# Установка необходимых пакетов для SSH
RUN apt-get update && apt-get install -y \
    openssh-server sudo gettext-base

# Настраиваем SSH-сервер
RUN mkdir /var/run/sshd

# Задаем пароль для пользователя postgres
RUN echo "postgres:123" | chpasswd

# Добавляем пользователя postgres в группу sudo
RUN usermod -aG sudo postgres

# Разрешаем вход с пустым паролем
RUN sed -i 's/#PermitEmptyPasswords no/PermitEmptyPasswords yes/' /etc/ssh/sshd_config

# Разрешаем аутентификацию по паролю
RUN sed -i 's/#PasswordAuthentication yes/PasswordAuthentication yes/' /etc/ssh/sshd_config


# Копируем SQL-скрипт для создания таблиц
COPY init.sql /docker-entrypoint-initdb.d/init.sql.raw


RUN mkdir -p /oracle/pg_data/archive 
RUN chmod 777 /oracle/pg_data/archive
RUN chown -R postgres:postgres /oracle/pg_data/archive
RUN mkdir -p /logs
RUN mkdir -p /etc/postgresql/
RUN service ssh start

ENTRYPOINT [ "bash", "-c", "\
mkdir -p /var/log/postgresql/ \n\
chown postgres:postgres /var/log/postgresql/ \n\
envsubst < /docker-entrypoint-initdb.d/init.sql.raw > /docker-entrypoint-initdb.d/init.sql \n\
cat <<EOF > /etc/postgresql/postgresql.conf \n\
listen_addresses = '*' \n\
log_destination = 'stderr' \n\
logging_collector = on \n\
log_directory = '/var/log/postgresql/' \n\
log_filename = 'postgresql.log' \n\
archive_mode = on \n\
archive_command = 'cp %p /oracle/pg_data/archive/%f' \n\
max_wal_senders = 10 \n\
wal_level = replica \n\
wal_log_hints = on \n\
log_replication_commands = on \n\
EOF\n\
cat <<EOF > /etc/postgresql/pg_hba.conf \n\
local all ${POSTGRES_USER} peer \n\
host all all 0.0.0.0/0 password \n\
host replication ${DB_REPL_USER} ${DB_REPL_HOST} trust \n\
host replication ${DB_REPL_USER} 172.18.0.0/24 trust \n\
host all all 172.18.0.1/32 trust \n\
host all all 0.0.0.0/0 trust \n\
EOF\n\
docker-entrypoint.sh $@"]


EXPOSE 22 5432

CMD ["postgres", "-c", "config_file=/etc/postgresql/postgresql.conf", "-c", "hba_file=/etc/postgresql/pg_hba.conf"]