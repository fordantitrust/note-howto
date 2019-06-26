# Hide Apache and PHP Information in HTTP Header

## Basic configuraion
### Apache
- `~$ vi /etc/apache2/conf-enabled/security.conf`
- Modify: `ServerTokens ProductOnly`
- Modify: `ServerSignature Off`

### PHP 
- `~$ vi /etc/php/5.6/fpm/php.ini`
- Modify: `expose_php Off`

## Advanced configuraion
- `~$ a2enmod security2`
- `~$ a2enmod headers`
- `~$ vi /etc/apache2/mods-enabled/security2.conf`
- Add: `SecServerSignature "<Some Information>"`
- `~$ vi /etc/apache2/conf-enabled/security.conf`
- Modify: `ServerTokens Full`
- Modify: `ServerSignature Off`
- `~$ vi /etc/apache2/sites-enabled/000-default.conf`
- Add: `Header always unset "X-Powered-By"`
- Add: `Header unset "X-Powered-By"`
- `~$ vi /etc/php/<Version>/fpm/php.ini`
- Modify: `expose_php Off`