sudo iptables -t nat -v -L PREROUTING -n --line-number

sudo sysctl net.ipv4.ip_forward=1

sudo iptables -t nat -A PREROUTING -i enp0s8 -p tcp --dport 80 -m state --state NEW -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.33.21:32038
sudo iptables -t nat -A PREROUTING -i enp0s8 -p tcp --dport 80 -j DNAT --to-destination 192.168.33.22:32038


sudo iptables -t nat -A PREROUTING -i enp0s9 -p tcp --dport 80 -m state --state NEW -m statistic --mode nth --every 2 --packet 0 -j DNAT --to-destination 192.168.33.21:32038
sudo iptables -t nat -A PREROUTING -i enp0s9 -p tcp --dport 80 -j DNAT --to-destination 192.168.33.22:32038


sudo iptables -t nat -A POSTROUTING -j MASQUERADE


borrar -> sudo iptables -t nat -D PREROUTING 1