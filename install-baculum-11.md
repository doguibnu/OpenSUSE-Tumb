## Como instalar o Baculum no OpenSUSE Tumbleweed:

Criar um diretório temporário para o Baculum:

    mkdir baculumtemp

Mudar para o diretório criado:

    cd /baculumtemp

Download do Baculum:

    wget --no-check-certificate https://sourceforge.net/projects/bacula/files/bacula/11.0.6/bacula-gui-11.0.6.tar.gz

Descompactar o arquivo:

    tar -xvf bacula-gui-11.0.6.tar.gz

Mudar para o diretório do arquivo descompactado:

    cd bacula-gui-11.0.6 

    Preparar o DESTINO temporário e construção de arquivos. Utilizar o comando:
    make build DESTDIR=/tmp/baculum-files WWWDIR=/srv/www/htdocs/baculum

Então, seguindo o exemplo acima:

    make build DESTDIR=/baculumtemp/bacula-gui-11.0.6/baculum WWWDIR=/srv/www/htdocs/baculum 

Dentro do diretório **/baculumtemp/bacula-gui-11.0.6/baculum** listando seu conteúdo, irá perceber que terá um novo **subdiretório: srv**, então:
Copiar os arquivos do tipo web para o diretório web do OpenSUSE:

    cp -R srv/www/htdocs/baculum/ /srv/www/htdocs/

Copiar os arquivos de configuração web do Apache para /etc/apache2/conf.d/:

    cp /baculumtemp/bacula-gui-11.0.6/baculum/etc/httpd/conf.d/baculum-*conf /etc/apache2/conf.d/
    
    
Copiar os arquivos básicos de autenticação:

API:

    cp /baculumtemp/bacula-gui-11.0.6/baculum/etc/baculum/Config-api-apache/baculum.users /srv/www/htdocs/baculum/protected/API/Config

Web:

    cp /baculumtemp/bacula-gui-11.0.6/baculum/etc/baculum/Config-web-apache/baculum.users /srv/www/htdocs/baculum/protected/Web/Config    
    
Copiar os arquivos de localização

English API:

    cp --remove-destination /baculumtemp/bacula-gui-11.0.6/baculum/usr/share/locale/en/LC_MESSAGES/baculum-api.mo /srv/www/htdocs/baculum/protected/API/Lang/en/messages.mo

Pl API:

    cp --remove-destination /baculumtemp/bacula-gui-11.0.6/baculum/usr/share/locale/pl/LC_MESSAGES/baculum-api.mo /srv/www/htdocs/baculum/protected/API/Lang/pl/messages.mo

PT API:

    cp --remove-destination /baculumtemp/bacula-gui-11.0.6/baculum/usr/share/locale/pt/LC_MESSAGES/baculum-api.mo /srv/www/htdocs/baculum/protected/API/Lang/pt/messages.mo

English Web:

    cp --remove-destination /baculumtemp/bacula-gui-11.0.6/baculum/usr/share/locale/en/LC_MESSAGES/baculum-web.mo /srv/www/htdocs/baculum/protected/Web/Lang/en/messages.mo

PL Web:

    cp --remove-destination /baculumtemp/bacula-gui-11.0.6/baculum/usr/share/locale/pl/LC_MESSAGES/baculum-web.mo /srv/www/htdocs/baculum/protected/Web/Lang/pl/messages.mo

Pt Web:

    cp --remove-destination /baculumtemp/bacula-gui-11.0.6/baculum/usr/share/locale/pt/LC_MESSAGES/baculum-web.mo /srv/www/htdocs/baculum/protected/Web/Lang/pt/messages.mo

Ja Web:

    cp --remove-destination /baculumtemp/bacula-gui-11.0.6/baculum/usr/share/locale/ja/LC_MESSAGES/baculum-web.mo /srv/www/htdocs/baculum/protected/Web/Lang/ja/messages.mo


Setar o proprietário e grupo dos arquivos copiados:
```
chown -R wwwrun:wwwrun /srv/www/htdocs/baculum/
```

Mudar para o diretório /baculumtemp/bacula-gui-11.0.6/baculum/etc com o comando:
```
cd /baculumtemp/bacula-gui-11.0.6/baculum/etc
```

Copiar Recursivamente o subdiretório baculun para **/etc**:
```
cp -r baculum /etc/
```

Mudar a permissão do grupo recursivamente para leitura, escrita e execução:
```
chmod -R g+rwx /etc/baculum/
```

Mudar o propritário do subdiretório **/etc/baculum/**:
```
chown -R wwwrun:bacula /etc/baculum/
```

Criar um grupo bacula:

    groupadd bacula

Políticas de segurança para o sudo não requisitar senha dos seguintes caminhos:

    echo "Defaults:wwwrun "'!'"requiretty
    wwwrun ALL=NOPASSWD:  /usr/sbin/bconsole
    wwwrun ALL=NOPASSWD:  /usr/sbin/bdirjson
    wwwrun ALL=NOPASSWD:  /usr/sbin/bsdjson
    wwwrun ALL=NOPASSWD:  /usr/sbin/bfdjson
    wwwrun ALL=NOPASSWD:  /usr/sbin/bbconsjson
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl start bacula-dir
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl stop bacula-dir
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl restart bacula-dir
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl start bacula-sd
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl stop bacula-sd
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl restart bacula-sd
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl start bacula-fd
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl stop bacula-fd
    wwwrun ALL=(root) NOPASSWD: /usr/bin/systemctl restart bacula-fd
    " > /etc/sudoers.d/baculum

Fazer o bacula pertencer ao grupo wwwrun:

    usermod -aG bacula wwwrun

Mudar o dono e o Grupo recursivamente dos diretórios:

    chown -R wwwrun:bacula /opt/bacula/working /opt/bacula/etc

Prover permissão de leitura, escrita e execução para o grupo no diretório (rwx):

    chmod -R g+rwx /opt/bacula/working /opt/bacula/etc

Ativar o modo rewrite do apache:

    a2enmod rewrite

Criar um diretório em /var/log/apache2/apierror e /var/log/apache2/apierrorlog:

    mkdir /var/log/apache2/apierror
    mkdir /var/log/apache2/apierrorlog

Editar o arquivo /etc/apache2/conf.d/baculum-api.conf e alterar as linhas:

    nano /etc/apache2/conf.d/baculum-api.conf
    CustomLog /var/log/apache2/apierror/baculum-api-access.log combined
    ErrorLog /var/log/apache2/apierrorlog/baculum-api-error.log

salvar o arquivo
Criar um diretório em /var/log/apache2/weberror e /var/log/apache2/weberrorlog:

    mkdir /var/log/apache2/weberror
    mkdir /var/log/apache2/weberrorlog

Editar o arquivo /etc/apache2/conf.d/baculum-web.conf e alterar as linhas:

    nano /etc/apache2/conf.d/baculum-web.conf
    CustomLog /var/log/apache2/weberror/baculum-web-access.log combined
    ErrorLog /var/log/apache2/weberrorlog/baculum-web-error.log

salvar o arquivo
Abrir portas 9095 e 9096 no firewall e recarregar firewall:

    firewall-cmd --permanent --zone=public --add-port=9095-9096/tcp
    firewall-cmd --reload

Reiniciar o sistema:

    reboot

Para acessar a API do Baculum:
Abrir um navegador:

    http://IP ou localhost:9096
    
    user: admin
    senha: admin

Para acessar o Baculum Web:
Abrir um navegador:

    http://IP ou localhost:9095
    
    user: admin
    senha: admin
 
 
 Fonte da instalação manual, acesse: [Baculum](https://baculum.app/doc/brief/installation.html#manual-installation)

Obrigado!
