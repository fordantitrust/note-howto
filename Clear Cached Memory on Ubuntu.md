# Clear Cached Memory on Ubuntu
`free && sync && echo 3 > /proc/sys/vm/drop_caches && free`