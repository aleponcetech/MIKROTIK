Você deve usar a senha secreta e pode usar mais e mais forte. E este artigo usará apenas a linha de comando - ele pode "traduzir" para qualquer GUI que você usar, seja a interface da web ou o Winbox.
Primeiro, crie um pool de endereços que os clientes VPN obterão uma vez conectados:

/ip pool add name=pool-vpn ranges=10.0.0.80-10.0.0.85

Permita que o L2TP / IPSec passe pela interface WAN. Certifique-se de que essas regras estejam acima das regras de firewall que bloqueiam todo o tráfego na interface WAN:

/ip firewall filter
add chain=input action=accept comment="VPN L2TP UDP 500" in-interface=wan protocol=udp dst-port=500 
add chain=input action=accept comment="VPN L2TP UDP 1701" in-interface=wan protocol=udp dst-port=1701
add chain=input action=accept comment="VPN L2TP 4500" in-interface=wan protocol=udp dst-port=4500
add chain=input action=accept comment="VPN L2TP ESP" in-interface=wan protocol=ipsec-esp
add chain=input action=accept comment="VPN L2TP AH" in-interface=wan protocol=ipsec-ah

Crie um perfil VPN que determinará os endereços IP do roteador, clientes VPN e servidor DNS. Você pode configurá-lo para ficar fora da sub-rede local, mas certifique-se de que seu firewall permita a conexão:

/ppp profile add change-tcp-mss=yes local-address=10.0.0.1 name=vpn-profile remote-address=pool-vpn dns-server=10.0.0.1 use-encryption=yes

Agora podemos criar usuários VPN:

/ppp secret add name="user" password="password" profile=vpn-profile service=any

Configure as opções IPSec, ou seja, padrões de criptografia, sigilo L2TP, conectável, NAT traversal:

/ip ipsec peer add address=0.0.0.0/0 exchange-mode=main-l2tp nat-traversal=yes generate-policy=port-override secret="sul2tpsecreto" enc-algorithm=aes-128,3des
/ip ipsec proposal set [ find default=yes ] enc-algorithms=aes-128-cbc,3des

Agora que tudo está pronto, podemos simplesmente habilitar o servidor VPN e escolher o perfil apropriado:

/interface l2tp-server server set authentication=mschap2 default-profile=vpn-profile enabled=yes max-mru=1460 max-mtu=1460 use-ipsec=yes

