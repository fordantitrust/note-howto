# Update GeoIP or GeoIP2 on gdnsd 

## Update GeoIP on gdnsd 

`wget http://geolite.maxmind.com/download/geoip/database/GeoLiteCountry/GeoIP.dat.gz`

`wget http://geolite.maxmind.com/download/geoip/database/GeoIPv6.dat.gz`

`gunzip GeoIP.dat.gz`

`gunzip GeoIPv6.dat.gz`

`mv GeoIP.dat /usr/share/GeoIP/`

`mv GeoIPv6.dat /usr/share/GeoIP/`

`cd /usr/share/GeoIP/`

`service gdnsd restart`

## Update GeoIP2 on gdnsd 

`wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.tar.gz`

`tar zxvf GeoLite2-Country.tar.gz`

`cd GeoLite2-Country_20180501`

`mv GeoLite2-Country.mmdb /usr/share/GeoIP/`

`service gdnsd restart`