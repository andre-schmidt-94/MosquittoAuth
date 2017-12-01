# MosquittoAuth

PREPARANDO O LINUX PARA A INSTALAÇÃO

Instale algumas bibliotecas necessárias:

sudo apt-get update
sudo apt-get install build-essential python quilt devscripts python-setuptools python3
sudo apt-get install libssl-dev
sudo apt-get install cmake
sudo apt-get install libc-ares-dev
sudo apt-get install uuid-dev
sudo apt-get install daemon
   sudo apt-get install build-essential libwrap0-dev libssl-dev libc-ares-dev uuid-dev xsltproc
	sudo apt-get install libssl-dev cmake libc-ares-dev uuid-dev daemon build-essential libwrap0-dev libssl-dev libc-ares-dev uuid-dev xsltproc python quilt devscripts python-setuptools python3
INSTALAÇÃO DO MOSQUITTO

Criar usuário “Mosquitto”

Você deve iniciar criando um usuário Mosquitto.
	sudo adduser mosquitto 
	cd /home/mosquitto

Instalação

	sudo wget http://mosquitto.org/files/source/mosquitto-1.4.9.tar.gz
	sudo tar -xvzf mosquitto-1.4.9.tar.gz
	cd mosquitto-1.4.9
	sudo make
	sudo make install

Set-up

O mosquitto já esta instalado, agora precisamos ajustar as configurações.

	sudo mosquitto_passwd -c /etc/mosquitto/pwfile ecomfort

O usuário e senha Default: ecomfort, Pixelti@1700.

	sudo mkdir /var/lib/mosquitto/
	sudo chown mosquitto /var/lib/mosquitto/ -R
	sudo cp /etc/mosquitto/mosquitto.conf.example /etc/mosquitto/mosquitto.conf

Ao copiar a configuração para a pasta /etc edite ela com as seguintes informações:
	sudo gedit /etc/mosquitto/mosquitto.conf

listener 1883 <yourIP>
persistence true
persistence_location /var/lib/mosquitto/
persistence_file mosquitto.db
log_dest syslog
log_dest stdout
log_dest topic
log_type error
log_type warning
log_type notice
log_type information
connection_messages true
log_timestamp true
allow_anonymous false
password_file /etc/mosquitto/pwfile

Por último, atualize os caminhos com o comando:
	sudo /sbin/ldconfig

Rodar/Testar o Mosquitto

Para rodar o servidor use o comando:
	mosquitto -c /etc/mosquitto/mosquitto.conf

Subscriber
	mosquitto_sub -h 127.0.0.1 -p 1883 -v -t '#' -u ecomfort -P Pixelti@1700

Publisher
	mosquitto_pub -h 127.0.0.1 -p 1883 -m 'mensagem teste' -t 'Gtw001' -u ecomfort -P Pixelti@1700





INSTALAÇÃO DO AUTENTICADOR VIA BD

	Antes de continuar com esse passo, certifique que o MySQL esteja instalado e rodando no seu computador. 

Instalação do OpenSSL
Para instalar siga os comandos:
	wget http://www.openssl.org/source/openssl-1.0.1u.tar.gz
	sudo tar -xvzf openssl-1.0.1u.tar.gz
	cd openssl-1.0.1u/
	./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl
	sudo make
	sudo make install


Configuração do mosquitto-auth-plug
Abra a configuração e copie:
		git clone https://github.com/jpmens/mosquitto-auth-plug.git

	cd /home/mosquitto/mosquitto-auth-plug-master
	gedit config.mk
	
# Select your backends from this list
BACKEND_CDB ?= no
BACKEND_MYSQL ?= yes
BACKEND_SQLITE ?= no
BACKEND_REDIS ?= no
BACKEND_POSTGRES ?= no
BACKEND_LDAP ?= no
BACKEND_HTTP ?= no
BACKEND_JWT ?= no
BACKEND_MONGO ?= no

# Specify the path to the Mosquitto sources here
MOSQUITTO_SRC = /home/mosquitto/mosquitto-1.4.9

# Specify the path the OpenSSL here
OPENSSLDIR = /usr/local/ssl

Antes de executar o comando ‘make’ deve-se executar alguns comandos para que não haja erros na compilação:
	sudo apt-get install libmysqlclient-dev
	sudo apt-get install libmariadbclient-dev

Com essas bibliotecas instaladas mude uma linha dentro do arquivo Makefile:
	sudo gedit Makefile

OSSLIBS = -L$(OPENSSLDIR)/lib -lcrypto -ldl

	sudo make

Por último volte na pasta do mosquitto-1.4.9, e faça uma alteração na linha:
	cd /home/mosquitto/mosquitto-1.4.9/
	sudo gedit config.mk

# Build with SRV lookup support.
WITH_SRV:=no


Sua mosquitto.conf deve ficar assim:
	
listener 1883 127.0.0.1
persistence true
persistence_location /var/lib/mosquitto/
persistence_file mosquitto.db
log_dest syslog
log_dest stdout
log_dest topic
log_type error
log_type warning
log_type notice
log_type information
connection_messages true
log_timestamp true
allow_anonymous false
password_file /etc/mosquitto/pwfile

auth_plugin /home/mosquitto/mosquitto-auth-plug/auth-plug.so
auth_opt_backends mysql
#auth_opt_redis_host 127.0.0.1
#auth_opt_redis_port 12885
auth_opt_host 127.0.0.1
auth_opt_port 3306
auth_opt_dbname pixel
auth_opt_user root
auth_opt_pass root
auth_opt_userquery SELECT pw FROM users WHERE username = '%s'
auth_opt_superquery SELECT COUNT(*) FROM users WHERE username = '%s' AND super = 1
auth_opt_aclquery SELECT topic FROM acls WHERE (username = '%s') AND (rw >= '%d')
auth_opt_anonusername AnonymouS


Server
mosquitto -c /etc/mosquitto/mosquitto.conf

Sub
mosquitto_sub -h 10.0.0.175 -p 1883 -t "#" -v -u MqttServer -P root

Pub
mosquitto_pub -h 10.0.0.175 -p 1883 -t "Gtw001" -m "teste" -u Gtw001 -P pixelti
