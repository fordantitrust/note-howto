# Block IP via iptables
## ADD Block IP (Class)
`iptables -A INPUT -s xxx.xxx.xxx.xxx/24 -j DROP`
## ADD Block IP
`iptables -I INPUT -s xxx.xxx.xxx.xxx -j DROP`
## REMOVE Block IP
`iptables -D INPUT -s xxx.xxx.xxx.xxx -j DROP`