# Create a LAMP stack (https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-lamp-stack)

1) Create a RG
```
az group create --name myResourceGroup --location eastus
```
2) Create a VM
```
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```
Output:
```
{
  "fqdns": "",
  "id": "/subscriptions/15aab67a-cc2e-4200-89d3-d2ad2e8bd399/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-22-48-2E-B5-F2",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "20.124.238.125",
  "resourceGroup": "myResourceGroup",
  "zones": ""
}
```

3) Open port 80 for web traffic
```
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

4) SSH into your VM
```
az network public-ip list --resource-group myResourceGroup --query [].ipAddress

ssh azureuser@40.68.254.142
```

5) Install Apache, MySQL, and PHP
```
sudo apt update && sudo apt install lamp-server^
```

6) Verify Apache
```
apache2 -v
```

7) Verify and secure MySQL
```
mysql -V

sudo mysql_secure_installation
```
- Set password
- Then hit 'y' for remaining settings
- `sudo mysql -u root -p` and enter password to verify connection to sql
- Then enter `\q` to exit

8) Verify PHP
```
php -v

sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
```
- Now you can check the PHP info page you created. Open a browser and go to http://yourPublicIPAddress/info.php

9) Install the WordPress package
```
sudo apt install wordpress
```

10) Configure Word Press
- In a working directory, create a text file wordpress.sql `sudo sensible-editor wordpress.sql`
- Enter the following into wordpress.sql:
```
CREATE DATABASE wordpress;
GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
ON wordpress.*
TO wordpress@localhost
IDENTIFIED BY 'yourPassword';
```
- Run the following to create a DB: `cat wordpress.sql | sudo mysql --defaults-extra-file=/etc/mysql/debian.cnf`
- Because wordpress.sql contains a pw, delete after use: `sudo rm wordpress.sql`
- Configure PHP: `sudo sensible-editor /etc/wordpress/config-localhost.php`
- Insert the following lines, replace password:
```
<?php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'yourPassword');
define('DB_HOST', 'localhost');
define('WP_CONTENT_DIR', '/usr/share/wordpress/wp-content');
?>
```
- Move WordPress to web server document root:
```
 sudo ln -s /usr/share/wordpress /var/www/html/wordpress

sudo mv /etc/wordpress/config-localhost.php /etc/wordpress/config-default.php
```
- Now you can complete the WordPress setup and publish on the platform. Open a browser and go to http://yourPublicIPAddress/wordpress.
  


