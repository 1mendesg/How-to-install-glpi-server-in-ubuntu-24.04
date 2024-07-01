# How-to-install-glpi-server-in-ubuntu-24.04
#!/bin/bash

# Configurações iniciais
GLPI_VERSION="10.0.15"
GLPI_TAR="glpi-${GLPI_VERSION}.tgz"
GLPI_URL="https://github.com/glpi-project/glpi/releases/download/${GLPI_VERSION}/${GLPI_TAR}"
GLPI_DIR="/var/www/glpi"

DB_NAME="glpidb"
DB_USER="glpiuser"
DB_PASSWORD="SuaSenhaSegura"

# Atualiza o sistema e instala as dependências necessárias
sudo apt update
sudo apt upgrade -y
sudo apt install -y apache2 php php-{apcu,cli,curl,gd,imap,intl,ldap,mysql,xml,zip,mbstring} mariadb-server wget tar

# Baixa e extrai o GLPI
cd /tmp
wget $GLPI_URL
tar -xvzf $GLPI_TAR
sudo mv glpi $GLPI_DIR

# Configura as permissões
sudo chown -R www-data:www-data $GLPI_DIR
sudo chmod -R 755 $GLPI_DIR

# Configura o Apache
sudo bash -c 'cat <<EOL > /etc/apache2/sites-available/glpi.conf
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/glpi

    <Directory /var/www/glpi>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog \${APACHE_LOG_DIR}/error.log
    CustomLog \${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
EOL'

sudo a2ensite glpi
sudo a2enmod rewrite
sudo systemctl reload apache2

# Configura o MySQL/MariaDB
sudo mysql -e "CREATE DATABASE ${DB_NAME} CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
sudo mysql -e "CREATE USER '${DB_USER}'@'localhost' IDENTIFIED BY '${DB_PASSWORD}';"
sudo mysql -e "GRANT ALL PRIVILEGES ON ${DB_NAME}.* TO '${DB_USER}'@'localhost';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Configurações adicionais do PHP
sudo bash -c 'echo "date.timezone = America/Sao_Paulo" >> /etc/php/7.4/apache2/php.ini'
sudo systemctl restart apache2

# Finaliza a configuração
echo "Instalação do GLPI finalizada."
echo "Acesse o GLPI pelo navegador em http://localhost/glpi"
echo "Use as credenciais padrão para login: usuário 'glpi' e senha 'glpi'"
