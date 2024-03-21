```
Varios arquivos tão aí no repositório, é só executar o comando:

git clone https://github.com/andrelps1/files

que ele vai baixar uma pasta no seu servidor chamada 'files', lá dentro tem os arquivos.
executa esse comando depois de atualizar o sistema.
```


```
<iface>     nome da sua interface.      ex: <iface>
<host>      nome da sua maquina.        ex: srv-xpto
<domain>    seu domínio.                ex: xpto.local
```


### Atualização dos pacotes do sistema
```bash
apt update
apt upgrade -y
```

<br>

### Configuração do arquivo `/etc/hosts`
Edite o arquivo `/etc/hosts` com o seguinte conteúdo:
```bash
vi /etc/hosts
```

#### Conteúdo do arquivo:
```bash
127.0.0.1       localhost
127.0.1.1       <host>
192.168.15.254   <host>.<domain>    <host>
```

<br>

### Configuração das interfaces de rede

```bash
vi /etc/network/interfaces
```

#### Conteúdo do arquivo:
```bash
# Interface de rede externa
auto eno1
iface eno1 inet dhcp

# Interface de rede interna
auto <iface>
iface <iface> inet static
        address 192.168.15.254
        netmask 255.255.255.0
        network 192.168.15.0
        broadcast 192.168.15.255
        dns-nameservers 192.168.15.254 8.8.8.8 1.1.1.1
        dns-search <domain>
        dns-domain <domain>
```

#### reinicie o serviço networking
```bash
systemctl restart networking
```
> da um `ip a` aí pra ver se a <iface> ta com o ip certo

<br>

### Configuração manual de resolução de DNS
```bash
systemctl disable --now systemd-resolved
unlink /etc/resolv.conf
vi /etc/resolv.conf
```

#### Conteúdo do arquivo:

```bash
# Endereço do controlador de domínio Samba
nameserver 192.168.15.254
# Endereço de fallback
nameserver 8.8.8.8
# Domínio de pesquisa para resolução de nomes
search <domain>
```

#### Adicione o atributo de imutabilidade:

```bash
chattr +i /etc/resolv.conf
```

<br><br>

## Firewall Iptables

### Arquivo de script das regras do firewall

#### conteúdo do arquivo `firewall.sh`:
```
O arquivo do firewall ta nas pastas do repositório aí
Tem que editar o arquivo do firewall e trocar os IP's e a interface de acordo com o seu servidor.

se quiser baixar os arquivos direto no servidor é só digitar o comando:

git clone https://github.com/andrelps1/files
```

#### Tornar o script um executavel
```bash
chmod +x firewall.sh
```

#### Mover o script para uma pasta do $PATH
```bash
mv firewall.sh /usr/local/sbin
```

#### Criar o arquivo de serviço para inicialização automática
```bash
vi /lib/systemd/system/firewall.service
```

#### conteúdo do arquivo:
```bash
[Unit]
Description=Inicializa o script firewall automaticamente.
After=network.target

[Install]
WantedBy=multi-user.target

[Service]
Type=oneshot
ExecStart=/usr/local/sbin/firewall.sh start
ExecStop=/usr/local/sbin/firewall.sh stop
RemainAfterExit=true
KillMode=process
```

#### Recarregar daemons do sistema
```bash
systemctl daemon-reload
```

#### Ativar o serviço do firewall
```bash
systemctl enable firewall.service
```

<br><br>

## Proxy Squid

### Instalar o `squid3` e `apache2`
```bash
apt install squid3 apache2
```

<br>

### Configuração do `squid`
```bash
vi /etc/squid/squid.conf
```

#### conteúdo do arquivo:
```
Também ta na pasta do repositório.
Tem que editar o arquivo do squid e colocar sua regra de sites proibidos e trocar os IP's caso o seu seja diferente, isso aí você mesmo faz.
```

#### Pasta de cache do `squid`
```bash
# Criar a pasta cache
mkdir /etc/squid/cache
# Dar permissão total
chmod 777 /etc/squid/cache
# Parar o serviço do Squid
service squid stop
# criar o banco de dados de cache do Squid
squid -z
```

#### Reiniciar o `squid`
```bash
service squid restart

squid -k parse # veja se não tem nenhum 'error' ou falha
squid -k reconfigure
```

## Sarg

```bash
# O sarg é tranquilo po faz aí
```

<br><br>

## DHCP

### Instalar o isc-dhcp-server
```bash
apt install isc-dhcp-server
```

<br>

### Configurar o arquivo /etc/dhcp/dhcpd.conf
conteúdo do arquivo:
```bash
# Configuração do serviço DHCP.
authoritative;

subnet 192.168.15.0 netmask 255.255.255.0 {
        range 192.168.15.10 192.168.15.200;
        option domain-name-servers 192.168.15.254, 8.8.8.8, 1.1.1.1;
        option domain-name "<host>.<domain>";
        option subnet-mask 255.255.255.0;
        option routers 192.168.15.254;
        option broadcast-address 192.168.15.255;
        default-lease-time 600;
        max-lease-time 7200;
        }
```

#### Configurar interface para execução do DHCP
```bash
vi /etc/default/isc-dhcp-server
```
conteúdo do arquivo:
```bash
INTERFACES="<iface>"
```

#### Reiniciar o serviço isc-dhcp-server
```bash
service isc-dhcp-server restart
```

> Agora voce liga seu windows aí e testa tudo primeiro, vê se o DHCP ta chegando direito com os dns certos, configura o proxy e vê se a internet e o squid
tambem tao funcionando direito e se ta bloqueando o site, testa tudo pra ver se ta tudo tranquilo.

> Se tiver funcionando da um **reboot**, se nao tiver aí fudeu tem alguma coisa errada


<br><br>

## A MERDA DO AD AGORA

### Instalar as dependências

<br>

na pasta tem o arquivo dependencias.sh que é só você digitar `bash dependencias.sh` que ele instala o que tem no arquivo. Ou tem o comando aí abaixo que é o mesmo do arquivo.
```bash
apt install build-essential libacl1-dev libattr1-dev libblkid-dev libgnutls-dev libreadline-dev python-dev libpam0g-dev python-dnspython gdb pkg-config libpopt-dev libldap2-dev dnsutils libbsd-dev docbook-xsl libcups2-dev nfs-kernel-server samba samba-common smbclient cifs-utils samba-vfs-modules samba-testsuite samba-dbg acl attr samba-dsdb-modules cups cups-common cups-core-drivers nmap winbind smbclient libnss-winbind libpam-winbind python3 krb5-user krb5-config -y
```

#### Campos de configuração do KERBEROS
```bash
# Realm
<domain>
# Servers
<host>.<domain>
# ADM Server
<host>.<domain>
```

<br>

### Desabilitar serviços e ativar o `samba-ad-dc`
```bash
systemctl disable --now nmbd smbd winbind

systemctl unmask samba-ad-dc
systemctl enable samba-ad-dc
```

<br>

### Renomeando o arquivo `smb.conf`
```bash
mv /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

> Dá outro **reboot** aí não custa nada👍

<br>

### Provisionando o domínio
```bash
samba-tool domain provision --use-rfc2307 --use-xattr=yes --interactive

# Campo REALM
    <domain>
# Campo DOMAIN
    [o primeiro pedaço do dominio]
# Campo SERVER ROLE
    dc
# Campo DNS BACKEND
    SAMBA_INTERNAL
# Campo DNS FORWARDER
    8.8.8.8
# Campo ADMINISTRATOR PASSWORD
    [Bota sua senha]
```

<br>

#### Mover o krb5.conf gerado
```bash
mv /var/lib/samba/private/krb5.conf /etc/
```

### Iniciar o `samba-ad-dc` e reiniciar o servidor
```bash
systemctl start samba-ad-dc
reboot
```

```bash
# Quando reiniciar executa:
host -t A <domain>

# Tem que retornar um:
# <domain> has [algum IP]
# Se ele retornar algo como 'Não econtrado' ou algo assim
# É porque deu alguma merda com seu domínio
# Se deu certo então agradece que é um bom sinal
```

Agora sai do dominio que teu windows tiver aí se ele tiver em um antigo, deixa ele zerado, executa o **arquivo de registro** de novo só pra ter certeza, entra de novo no dominio e vê se deu certo, se der certo otimo, se nao der tu fez alguma coisa errada, ou eu fiz alguma coisa errada, ou a vida te odeia mesmo. boa sorte👍
