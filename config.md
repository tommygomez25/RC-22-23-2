# Configuration

## Reset configurations

1) **systemctl restart networking** ( se não funcionar dar reboot )
2) **system reset-configuration** no **GTK** (mudar **baudrate** para **115200**)

### Cabos 

1) Ligar cabo CISCO RS232 -> **switch** console (T4)
2) Ligar RS232 CISCO  -> s0 do tux4 (por exemplo) (T3)
3) Ligar restantes **e0** ao switch e ver em que **porta** estão : <br />
 e0(2) -> 8 <br />
 e0(3) -> 16 <br />
 e0(4) -> 24 <br />

### Reset
1) **ifconfig** para ver se existe alguma interface ligada
2) **ifconfig eth0 down** em todos os computadores (2,3,4)


## Experience 1

1) **ifconfig eth0 up** em todos os computadores (2,3,4)
2)  TUX 2- **ifconfig eth0 up 172.16.Y1.1/24**
3) TUX 3- **ifconfig eth0 up 172.16.Y0.1/24**
4) TUX 4- **ifconfig eth0 up 172.16.Y0.254/24**

## Experience 2

1) **/interface bridge print** no GTK para verificar bridges 
2) **/interface bridge add name=bridgeY0**
3) **/interface bridge add name=bridgeY1**
4) **/interface bridge port remove [find interface=ether8]**
5) **/interface bridge port remove [find interface=ether16]**
6) **/interface bridge port remove [find interface=ether24]**
7) **/interface bridge port add bridge=bridgeY0 interface=ether16** (TUX 3)
8) **/interface bridge port add bridge=bridgeY0 interface=ether24** (TUX 4)
9) **/interface bridge port add bridge=bridgeY1 interface=ether8** 

> **NOTA** : Se der ping do tux 3 para o tux 2, não funciona !!

## Experience 3

1) Ligar **e1 do tux 4** ao **ether12 do switch**
2) No tux 4 -> **ifconfig eth1 down**
3) **ifconfig eth1 172.16.Y1.253/24**
4) NO GTK -> **/interface bridge port remove [find interface=ether12]**
5) NO GTK -> **/interface bridge port add bridge=bridgeY1 interface=ether12**
6) No tux4 -> `Enable IP Forwarding` **echo 1 > /proc/sys/net/ipv4/ip_forward**
`Disable ICMP echo-ignore-broadcast` **echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_broadcasts**
7) Adicionar rotas de modo a que o tux3 e tux2 consigam comunicar <br />
8)  No tux 3-> **route add -net 172.16.Y1.0/24 gw 172.16.Y0.254**
9) No tux 2 -> **route add -net 172.16.Y0.0/24 gw 172.16.Y1.253**

> **NOTA** : Limpar arp tables é para limpar a cache das arps, de modo a aparecer nos logs do wireshark (**arp -a** para saber, e **arp -d `ipaddress`** para delete)

## Experience 4

1) Ligar cabo do ether1 do **router** ao PY.1 
2) Ligar cabo do ether2 do **router** a uma porta do switch -> porta 20
3) **/interface bridge port remove [find interface=ether20]**
4) **/interface bridge port add bridge=bridgeY1 interface=ether20**
5) Pôr o tux4 como router default do tux3 e pôr o router como router default do tux2 e tux4 
6) **Para configurar o router** -> tirar cabo da consola do switch e ligar ao **router MTIK**
7) **/ip address add address=172.16.Z.Y9/24 interface=ether1** (Z na I321 = 1, Z na I320 = 2)
8) **/ip address add address=172.16.Y1.254/24 interface=ether2**
9) No tux 3 -> **route add default gw 172.16.Y0.254**
10) No tux 2 e 4-> **route add default gw 172.16.Y1.254**
11) Ir ao GTK do tux4 (que é onde está ligado o **MicroTIK**) e adicionar rota para o **router**: 
**/ip route add dst-address=172.16.Y0.0/24 gateway=172.16.Y1.253**

11) Dar ping do tux 3 para todas as interfaces

12) No tux 2 -> **echo 0 > /proc/sys/net/ipv4/conf/eth0/accept_redirects**
13) No tux 2 -> **echo 0 > /proc/sys/net/ipv4/conf/all/accept_redirects**

13) **route delete -net 172.16.Y0.0/24 gw 172.16.Y1.253** (não é preciso para demonstração)
> **NOTA** : Agora o tux2 precisa de ir ao router para comunicar com o tux3, visto que já não tem a gateway do tux4 <br />
traceroute 172.16.Y0.1 (tux3) -> 3 hops <br />
Depois de adicionar a rota outra vez já so faz 2 hops

14) Para desativar NAT -> no GTK ->**/ip firewall nat disable 0** (não é preciso para demonstração)
<br />Conseguimos verificar que depois de dar disable ao NAT , já não há forma de passar de IP privado para IP público, pelo que o ping não funciona

## Experience 5

1) Em todos os tuxs correr o comando -> **nano /etc/resolv.conf** e verificar que tem lá o server
> **NOTA** : Ao dar ping a google.com não funciona porque o router não tem uma rota para qualquer outro IP além dos que adicionamos
2) **/ip route add dst-address = 0.0.0.0/0 gateway=172.16.Z.254** (Z na I321 = 1, Z na I320 = 2) <br />
> **NOTA** : Agora já é possível comunicar com qualquer outro ip
