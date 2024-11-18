# Neste tutorial, utilizarei um servidor Debian 11 para instalar o Novo SGA 2.0.8. No entanto, realizei testes com a versão 10 do Debian e também com as versões 20.04 e 22.04 do Ubuntu Server, e o processo funcionou perfeitamente em todas essas versões.

O Novo SGA é um sistema de gerenciamento de filas de atendimento desenvolvido para locais que atendem ao público. Com o Novo SGA, é possível configurar serviços, definir prioridades e gerenciar atendentes de forma personalizada, adaptando o sistema às necessidades específicas de cada organização.

O Novo SGA é mais do que um sistema de controle de filas. Ao gerenciar o fluxo de atendimento, o sistema apresenta uma série de recursos que auxiliam na gerência e administração das unidades de atendimento. [Fonte](https://suporte.4yousee.com.br/pt-BR/support/solutions/articles/72000535945-novo-sga-sistema-de-gerenciamento-de-atendimento).

Github da aplicação [clique aqui](https://github.com/novosga).

Site oficial do desenvolvedor [clique aqui](http://novosga.org/).

### Pré-requisitos:
- Apache 2;
- MariaDB 10.5 ou Mysql;
- PHP 7.4 e depêndencias.
Abordaremos a instalação dos mesmos neste tutorial.

***Nota: siga este tutorial em um ambiente de testes antes de aplicar em um ambiente de produção.***

### Instalação NovoSGA 2.0.8 no Debian server

***Nota**: é de suma importância que o servidor tenha uma quantidade razoável de memória RAM [ex.: acima de 2GB]*

Atualizamos o sistema

```bash
apt update && apt upgrade -y
```

Alterar a **data e hora** manualmente

```bash
# alterar timezone
timedatectl set-timezone America/Sao_Paulo
```

### Instale o servidor de banco de dados

Instale o **MariaDB** server 10.5

```bash
apt install mariadb-server -y
```

Habilite o serviço

```bash
systemctl status mariadb

systemctl start mariadb

systemctl enable mariadb
```

Script de inicialização segura do MariaDB

```bash
mysql_secure_installation
```
**Algumas perguntas serão feitas, você pode responder conforme a orientação abaixo:**

- se você tem uma senha de root do banco de dados digite, **se não**, tecle Enter.
- mudar para autenticação unix_socket: **Yes**
- para definir uma nova senha digite: **Yes**
- remover usuários anônimos: **Yes**
- desativar o login remoto do root: **Yes**
- remover o banco de dados teste: **Yes**
- recarregar todas as tabelas: **Yes**

Por fim, Você deve receber algo como: All done! Thanks for using MariaDB.

Acesse o banco de dados [***usuário root do banco***]

```bash
mysql -u root -p

# digite o senha do root do banco de dados
```

Vamos criar o **usuário** de acesso ao banco de dados e o Banco ***novosgaDB***

```sql

SHOW DATABASES;

CREATE DATABASE novosgaDB DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;

GRANT ALL PRIVILEGES ON novosgaDB.* TO 'usernovosga'@'localhost' IDENTIFIED BY 'SENHA1234';

FLUSH PRIVILEGES;

SHOW DATABASES;

SELECT user,host FROM mysql.user;

EXIT;
```
**Nota: Adicionar os dados de acordo com seu banco de dados.**

Instale **PHP 7.4** e suas dependências

Acesse

```bash
cd /var/opt/
```

### Instalação do repositório PHP e seus pacotes adicionais

```bash
apt install -y lsb-release ca-certificates apt-transport-https software-properties-common gnupg2

echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/sury-php.list

apt install curl vim wget unzip nano -y

curl -fsSL  https://packages.sury.org/php/apt.gpg| gpg --dearmor -o /etc/apt/trusted.gpg.d/sury-keyring.gpg

apt update
```

Instalar o PHP e módulos/bibliotecas

```bash
apt install php7.4 -y

apt install php7.4-cli php7.4-common php7.4-mysql php7.4-zip php7.4-gd php7.4-mbstring php7.4-curl php7.4-xml php7.4-bcmath php7.4-intl php7.4-ldap php7.4-bz2 -y
```

Saber a versão do PHP instalada e módulos

```bash
php -v

php --modules
```

### Instale o Apache

Ver se determinado programa está instalado e **instalar Apache** (***precisa estar como root***)

```bash
# Instale o Apache
apt install apache2
```

Vamos ativar um **módulo** do Apache para escrita, para isso digite

```bash
a2enmod env rewrite

# ver mods ativos
a2enmod
```

Verificar se as configurações do Apache estão **Ok**

```bash
apache2ctl configtest
```

Reinicie o Apache

```bash
systemctl restart apache2
```

Agora, vamos alterar o de configuração de segurança do Apache

```bash
cd /etc/apache2/
```

Crie uma cópia do arquivo

```bash
cp apache2.conf apache2.conf.bkp
```

Edite o arquivo

```bash
vim /etc/apache2/apache2.conf
```

Agora, vamos no arquivo de configuração do Apache

```bash
vim /etc/apache2/apache2.conf
```

![Altere](https://i.ibb.co/qm8sZPm/Untitled.png)

Altere de None para **All**

Salve e saia do arquivo.


### Iniciando a instalação do **Novo SGA 2.0.8**

Instale o **Composer v2.5** 

```bash
apt update

cd ~

curl -sS https://getcomposer.org/installer -o composer-setup.php

HASH=`curl -sS https://composer.github.io/installer.sig`

echo $HASH

php -r "if (hash_file('SHA384', 'composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

# verificando a instalação (temos que receber sucessfully)
php composer-setup.php --install-dir=/usr/local/bin --filename=composer

# Test your installation by running this command
composer
```

Baixe os pacotes do **Novo SGA 2.0.8**

```bash
cd ~

# aplicação
composer create-project "novosga/novosga:2.0.8" ~/novosga -vvv
```

Vamos editar o arquivo ***composer.json***

```sql
vim novosga/composer.json
```

Busque pelo parâmetro destacado na imagem abaixo e adicione o sinal de **^** ficando assim *^1.0* para podermos trabalhar com **versões superiores do composer**, veja

![Untitled](https://i.ibb.co/TqQj2B0/Untitled-1.png)

*Entrar na pasta do projeto e Atualizar via composer*

```bash
cd novosga

composer update -vvv

```

Caso algumas perguntas sejam feitas, digite o seguinte:
- Pergunta 1 digite: Y
- Pergunta 2 digite: Y
- Pergunta 3 digite: N

*Sair da pasta e mover o projeto pra dentro da pasta root do apache*

```bash
cd ..

mv novosga /var/www/html/
```

*Vá pra **/var/www/html/novosga**, torne executável o **bin/console** e faça o clean-up e warm-up de cache*

```bash
cd /var/www/html/novosga/

chmod +x bin/console

bin/console cache:clear --no-debug --no-warmup --env=prod -vv

bin/console cache:warmup --env=prod
```

*Criando o **.htaccess** contendo configs de rewrite e conexão ao BD na pasta **/var/www/html/novosga/public/*** 

***Adicionar os dados de acordo com seu banco de dados.***

```bash
nano /var/www/html/novosga/public/.htaccess

# Insira este conteúdo abaixo ao arquivo

Options -MultiViews
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^(.*)$ index.php [QSA,L]
SetEnv APP_ENV prod
SetEnv LANGUAGE pt_BR
SetEnv DATABASE_URL mysql://usernovosga:SENHA1234@localhost:3306/novosgaDB
```

*Export de variáveis de ambiente necessárias e comando install, estando **dentro** de **/var/www/html/novosga/***

```bash
cd /var/www/html/novosga/

export APP_ENV=prod LANGUAGE=pt_BR DATABASE_URL="mysql://usernovosga:SENHA1234@localhost:3306/novosgaDB"
```

**Executar a instalação do serviço**

```bash
bin/console novosga:install
```

**Algumas perguntas serão feitas:**
- a primeira é sobre o username do administrador, tecle enter para manter *admin*. 
- a segunda pergunta é para *definir uma senha para o admin*, digite a senha e tecle enter. 
- para as próximas perguntas basta teclar Enter, você poderá definir as nomenclaturas via interface web da aplicação.

***Dê a permissão de escrita, leitura e execução, além de alterar usuário e grupo dono***

```bash

# acesse o diretório
cd /var/www/html/

# usuário e grupo
chown -R www-data:www-data .

# permissões arquivos
find . -type f -exec chmod 640 {} \;

# permissões diretórios
find . -type d -exec chmod 750 {} \;
```

*Reinicie o apache para aplicar as configurações*

```bash
systemctl restart apache2
```

Agora, acesse o **php.ini**

```bash
cd /etc/php/7.4/apache2/

# gere um copia
cp php.ini php.ini.bkp

# acesse o arquivo
vim php.ini
```

Alterar valores no **PHP.ini (**Altere as configurações do arquivo de acordo com esses parâmetros abaixo**)** Procure pelas linhas e altere os parâmetros

```bash
388 max_execution_time = 50
398 max_input_time = 60
405 max_input_vars = 5000
409 memory_limit = 256M
465 error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
482 display_errors = Off
694 post_max_size = 8M
713 default_charset = "UTF-8"
837 file_uploads = On
846 upload_max_filesize = 5M
849 max_file_uploads = 20
962 date.timezone = 'America/Sao_Paulo'
```

Reinicie o Apache

```bash
systemctl restart apache2
```

Agora, **acesse o navegador** de internet para continuar a instalação do Novo SGA

Ex.: *http://ip-servidor/novosga/public/login*

- User: admin
- Senha: *a que você acabou de definir*

### Neste passo, vamos instalar o Painel pra chamada de Senhas

Acesse,

```bash
# acessando o diretório pra download
cd /var/opt/

# Baixe o painel
wget https://github.com/novosga/panel-app/releases/download/v2.0.1/painel-web-2.0.1.zip

# descompactando o arquivo
unzip painel-web-2.0.1

# alterando o nome
mv painel-web-2.0.1 painel-web

```

**Mova-o de local, movendo a pasta de diretório**

```bash
mv painel-web /var/www/html/novosga/public/

cd /var/www/html/novosga/public/
```

***Dê a permissão de escrita, leitura e execução, além de alterar usuário e grupo dono de Painel***

```bash
# usuário e grupo
chown -R www-data:www-data .

# permissões arquivos
find . -type f -exec chmod 640 {} \;

# permissões diretórios
find . -type d -exec chmod 750 {} \;
```

**Acesso ao Painel de chamamento de senhas**

*http://ip-servidor/novosga/public/painel-web/index.html*

**Após acesso ao painel, será preciso integrá-lo ao nosso SGA.**

Agora, via **navegador de internet** acesse a console de gerenciamento do Novo SGA:

Ex.: *http://ip-servidor/novosga/public/login*

![Untitled](https://i.ibb.co/B4gj4Rj/Untitled-5.png)

Precisaremos criar o Token de API pra autenticar nosso painel de senhas, acesse **Web API**

![Untitled](https://i.ibb.co/47gxtV7/Untitled-6.png)

![Untitled](https://i.ibb.co/YRgkMZG/Untitled-7.png)

Copie as credencias pra integração conforme imagem:

![Untitled](https://i.ibb.co/JCWnm8y/Untitled-8.png)

**Volte no Painel de Senhas** e cole as credencias copiadas e insira os dados do server (Local onde o SGA está hospedado), username e password do administrador do SGA.

![Fazendo a integração do painel com servidor](https://i.ibb.co/xGSL0mD/Untitled-9.png)

Definindo a unidade. Fazendo a integração do painel com servidor. **Selecione a unidade e seus serviços disponibilizados**.

![Definindo a unidade](https://i.ibb.co/cQQpJvn/Untitled-10.png)

Agora, clique em **voltar** no canto superior esquerdo para visualizar o Painel configurado.


### Resetar as senhas automaticamente via crontab
Caso precise você pode definir uma tarefa para zerar as senhas geradas no dia anterior. Isso é muito útil no dia a dia.

Dê permissão de execução do arquivo **novosga** com comando

```bash

cd /var/www/html/novosga/bin/

chmod 775 console

touch console novosga:reset

chmod +x /var/www/html/novosga/bin/console/novosga

ls -l

# teste o reset via terminal **Adicionar os dados de acordo com seu banco de dados.**

APP_ENV=prod LANGUAGE=pt_BR DATABASE_URL="mysql://usernovosga:SENHA1234@localhost:3306/novosgaDB" /var/www/html/novosga/bin/console novosga:reset
```

Cole ou digite o comando a seguir: **Adicionar os dados de acordo com seu banco de dados.**

```bash
crontab -e

# resetar as senhas do sga diariamente as 00h05

05 00 * * * APP_ENV=prod LANGUAGE=pt_BR DATABASE_URL="mysql://usernovosga:SENHA1234@localhost:3306/novosgaDB" /var/www/html/novosga/bin/console novosga:reset
```

Listar as tarefas agendadas

```bash
crontab -l
```
Agora na hora agendada as senhas serão zeradas.

### Alterar logo da tela de login Novo SGA

Via **FileZilla**, acesse seu servidor e busque pelo caminho abaixo

```bash
/var/www/html/novosga/public/images/
```

Faça uma cópia do arquivo original e utilize o mesmo nome e extensão para o novo arquivo de imagem.

Reinicie o servidor
```bash
reboot
```

### Então é isso, basta agora seguir com as configurações da aplicação via interface web. Espero ter te ajudado!

Acesse o navegador de internet para acessar o Novo SGA, ex.: http://ip-servidor/novosga/public/login

### ----

### Galera fiz um vídeo tutorial de instalação do Novo SGA seguindo este passo a passo, vejam:

Vídeo 01: https://youtu.be/xwZDpG-4jQo?si=airSD1sDbtZfpSeF

Vídeo 02: https://youtu.be/vi0SZ80uOe8?si=zczDOrxPc7fxPOeG
