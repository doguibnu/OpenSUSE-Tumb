# **Instalar Bacula no OpenSUSE Tumbleweed** #

Após a instalação do sistema é necessário inserir o nome de hostname no
arquivo **/etc/hostname**. Para isso, utilize o editor nano com o
comando: 
````
nano /etc/hostname
ecolha um nome para seu server por exemplo: baculasrv
````
salve o arquivo com o comando:
 ```
 control + o
 ```` 

e para sair do editor, com o comando:
````
contorl + x
````

Alterar o hostname sem necessidade de executar reboot no servidor. Use o
comando: 
````
echo “nome_escolhido” \> /proc/sys/kernel/hostname
````

Saia do sistema com o comando:
```
logout
```
Logue-se novamente e o nome
estará alterado.

Atualizar o sistem com o comando: 
```
zypper up
```


## Instalação dos pacotes necessários para compilar, instalar e rodar o Bacula.

````
zypper in -y make lzo-devel gcc gcc-c++ mingw64-zlib1 libssl52 mt-st mtx libacl-devel postgresql10-devel postgresql10-server
````
Abrir portas do Firewall para comunicação do Bacula e executar um Reload no Firewall
````
firewall-cmd --permanent --zone=public --add-port=9101-9103/tcp  
firewall-cmd --reload
````

**Iniciar** o Postgresql: 
````
systemctl start postgresql
````

Verificar o **Status** do serviço **Postgresql:**:
````
systemctl status postgresql
````

**Habilitar** o serviço para iniciar no boot do sistema:
````
systemctl enable postgresql
````

## **Fazer Download do Bacula (versão 11.06)**

Fazer um diretório para o bacula: 
````
mkdir baculatemp
````

Mudar para o diretório criado:
````
cd baculatemp
````

Fazer o download do arquivo de instalação do bacula com o comando:
````
wget -O bacula-11.0.6.tar.gz https://sourceforge.net/projects/bacula/files/bacula/11.0.6/bacula-11.0.6.tar.gz/download
````
Descompactar o arquivo baixado com o comando:
````
tar -zxvf nome_arquivo_tar.gz
````
Mude para o diretório onde foi descompactado o mesmo e então, fazer o
comando para compilar o bacula: Atenção para os parâmetros:
--with-job-email=**seu-e-mail-desejado** --with-hostname=**hostname-ou-IP-desejado**

``````
./configure --with-readline=/usr/include/readline --disable-conio --bindir=/usr/bin --sysconfdir=/opt/bacula/etc --sbindir=/usr/sbin --with-scriptdir=/opt/bacula/scripts --with-working-dir=/opt/bacula/working --with-logdir=/var/log–enable-smartalloc --with-postgresql --with-archivedir=/mnt/backup --with-job-email=e-mail_desejado --with-hostname=IP_desejado
````````

Após a finalização siga com os comandos:
````
make -j 8
````

E então:
````
make install
````

Deve gerar uma saída das opções de configuração da instalação

``````
Configuration on Wed Jul 27 14:22:40 -03 2022:

   Host:                      x86_64-pc-linux-gnu -- unknown unknown
   Bacula version:            Bacula 11.0.6 (10 March 2022)
   Source code location:      .
   Install binaries:          /usr/sbin
   Install libraries:         /usr/lib64
   Install config files:      /opt/bacula/etc
   Scripts directory:         /opt/bacula/scripts
   Archive directory:         /mnt/backup
   Working directory:         /opt/bacula/working
   PID directory:             /var/run
   Subsys directory:          /var/run/subsys
   Man directory:             /usr/share/man
   Data directory:            /usr/share
   Plugin directory:          /usr/lib64
   C Compiler:                gcc Linux)
   C++ Compiler:              /usr/bin/g++ Linux)
   Compiler flags:             -g -O2 -Wall -x c++ -fno-strict-aliasing -fno-exceptions -fno-rtti
   Linker flags:               
   Libraries:                 -lpthread 
   Statically Linked Tools:   no
   Statically Linked FD:      no
   Statically Linked SD:      no
   Statically Linked DIR:     no
   Statically Linked CONS:    no
   Database backends:         PostgreSQL
   Database port:              
   Database name:             bacula
   Database user:             bacula
   Database SSL options:      

   Job Output Email:          e-mail@dominio.com
   Traceback Email:           root@localhost
   SMTP Host Address:         localhost

   Director Port:             9101
   File daemon Port:          9102
   Storage daemon Port:       9103

   Director User:             
   Director Group:            
   Storage Daemon User:       
   Storage DaemonGroup:       
   File Daemon User:          
   File Daemon Group:         

   Large file support:        yes
   Bacula conio support:      no -lreadline -lhistory -ltinfo
   readline support:          yes 
   TCP Wrappers support:      no 
   TLS support:               yes
   Encryption support:        yes
   ZLIB support:              yes
   LZO support:               yes
   S3 support:                no
   enable-smartalloc:         yes
   enable-lockmgr:            no
   bat support:               no
   client-only:               no
   build-dird:                yes
   build-stored:              yes
   Plugin support:            yes
   AFS support:               no
   ACL support:               yes
   XATTR support:             yes
   GPFS support:              no 
   systemd support:           no 
   Batch insert enabled:      PostgreSQL

   Plugins:
   - Docker:                  no
```````


Criar um usuário **bacula** com a senha **bacula** no postgres. No terminal mudar para o usuário postgres:
````
su - postgres
````
Executar o comando:  
````
psql
````

Criar o usuário bacula com o password bacula com o comando:
````
CREATE USER bacula WITH PASSWORD 'bacula';
````
Para sair:
````
\q
exit
````

No Terminal mudar as permissões do diretório executando o comando:
````
chmod 775 /opt/bacula/
````

Mudar para o diretório **/opt/bacula/scripts**  
````
cd /opt/bacula/scripts
````

Mudar o proprietário para postgres dos seguintes arquivos com o
comando:
``````
chown postgres create_postgresql_database && chown postgres make_postgresql_tables  
chown postgres grant_postgresql_privileges && chown postgres drop_postgresql_database  
chown postgres update_postgresql_tables
``````

Trocar o usuário para postgres:  
````
su postgres
````

**Criar** a base de dados do postgresql,  
**Fazer** as tabelas,  
**Garantir** privilégios   
**Sair** com os comandos

Então: 
````
./create_postgresql_database 
./make_postgresql_tables
./grant_postgresql_privileges  
exit
``````
Editar o arquivo **/var/lib/pgsql/data/postgresql.conf**:  
````
nano /var/lib/pgsql/data/postgresql.conf 
````
Antes: 
**# listen_addresses = **‘localhost’**  

Para:  
````
listen_addresses = '*'
````

**De:**
#port = 5432**

**Para:**
````
port = 5432
````
Sair do arquivo


Editar o arquivo **/var/lib/pgsql/data/pg_hba.conf:**
````
nano /var/lib/pgsql/data/pg_hba.conf
``````
**Antes:**  
\# “local” is for Unix domain socket connections only  
local   all             all                                     peer  
\# IPv4 local connections:  
host    all             all             127.0.0.1/32            ident  
\# IPv6 local connections:  
host    all             all             ::1/128                 ident  
\# Allow replication connections from localhost, by a user with the  
\# replication privilege.
_______

**PARA:**
````
# "local" is for Unix domain socket connections only
local   all             all                                     md5
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
# Allow replication connections from localhost, by a user with the
# replication privilege.
``````
**Salvar o arquivo**


Editar o arquivo **bacula-dir.conf** e inserir as credenciais para se
conectar a porta **5432**:  
````
nano /opt/bacula/etc/bacula-dir.conf
Procurar pela linha Catalog e insira as linhas como abaixo:

Catalog {  
Name = MyCatalog  
dbdriver = "postgresql"; dbaddress = 127.0.0.1; dbport = 5432  
dbname = "bacula"; dbuser = "bacula"; dbpassword = "bacula"
}
`````
**Salvar o arquivo**


Abrir a porta **5432** no Firewall e recarregar o firewall:
````
firewall-cmd --permanent --zone=public --add-port=5432/tcp  
firewall-cmd --reload
````

**Fazer um arquivo Unit do serviço Bacula**

Mudar para o diretório:  
````
cd /etc/systemd/system
````
Criar o arquivo:
````
nano bacula.service
````

**Copie e cole com o seguinte conteúdo para seu arquivo:**
_____
`````
[Unit]  
Description=Bacula backup service  
After=syslog.target network.target

[Service]  
Type=forking  
ExecStart=/opt/bacula/scripts/bacula start  
ExecReload=/opt/bacula/scripts/bacula reload  
ExecStop=/opt/bacula/scripts/bacula stop

[Install]  
WantedBy=multi-user.target
``````
**Salvar o arquivo**
______

Recarregar o **daemon** do **systemd**: 
````
systemctl daemon-reload
````
**Iniciar** o Serviço bacula:
````
systemctl start bacula.service
````
Verificar o **Status** do serviço bacula:
````
systemctl status bacula.service
````
**Parar** o serviço do bacula: 
````
systemctl stop bacula.service
````

**Habilitar** o serviço bacula para iniciar com o sistema:
````
systemctl enable bacula.service
````
**Reiniciar** o servidor:
````
reboot
````

**Testando o Bacula**  
Lembrando que os serviços do **Postgresql** e **Bacula** devem estar
ativos. Verificar com os comandos:
````
systemctl status bacula.service  
systemctl status postgresq
````
Agora, no terminal chame o **bconsole**:
````
bconsole
Connecting to Director 10.1.1.23:9101
1000 OK: 10002 srvbacula-dir Version: 11.0.6 (10 March 2022)
Enter a period to cancel a command.
*
````

Se aparecer o * indica que o **bacula está funcional.**
