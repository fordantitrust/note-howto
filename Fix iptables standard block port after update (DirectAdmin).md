# Fix iptables standard to block port after update (DirectAdmin)

`cd /usr/libexec/iptables`

`mv iptables.init iptables.init.backup`

`wget -O iptables.init http://files.directadmin.com/services/all/block_ips/2.2/iptables`

`chmod 755 iptables.init`

`systemctl reload iptables`

Ref: [I wish to have a block_ip.sh so I can block IPs through DirectAdmin](https://help.directadmin.com/item.php?id=380 "I wish to have a block_ip.sh so I can block IPs through DirectAdmin")