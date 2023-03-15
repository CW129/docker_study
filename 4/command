# Network Namespace 생성

ip link add veth0 type veth peer name veth1
ip netns add RED
ip netns add BLUE
ip netns list

ip link set veth0 netns RED
ip link set veth1 netns BLUE
ip netns exec RED ip link set veth0 up
ip netns exec BLUE ip link set veth1 up
ip netns exec RED ip addr add 11.11.11.2/24 dev veth0
ip netns exec BLUE ip addr add 11.11.11.3/24 dev veth1

nsenter --net=/var/run/netns/RED



* virtual ethernet은 항상 pair로 생성
* /var/run/netns 에 영속



nsenter -t (process id) -n ip link
nsenter -t (process id) -a hostname

# IPVlan 생성
docker network create -d ipvlan --subnet 192.168.219.0/24 --gateway 192.168.219.1 -o parent=enp1s0 pub_ipvlan
docker network create -d macvlan --subnet 192.168.219.0/24 --gateway 192.168.219.1 -o parent=enp1s0 pub_mac
