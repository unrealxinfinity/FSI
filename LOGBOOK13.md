# Task 1.1A
> O objetivo foi perceber como a ferramenta para sniffing funcionava

Usando a livraria scapy, conseguimos criar a nossa própria ferramenta de sniffing pelo seguinte código python:

```py
#!/usr/bin/env python3
from scapy.all import *


def print_pkt(pkt):
  pkt.show()
print(get_if_list())
interfaces=['br-a7adb5b1b35b','enp0s3']
pkt = sniff(iface=interfaces, filter='icmp', prn=print_pkt)
```
## Passos
### 1. Setup do docker obtendo 3 containers;
![img](https://media.discordapp.net/attachments/1153998326274994216/1187135850413305876/image.png?ex=6595c96a&is=6583546a&hm=3e973b063ba133503a2a7da88d5499a9d0fc94962e02ea0967c92c36271d52e9&=&format=webp&quality=lossless&width=1440&height=515)

### 2. Obter a interface em que a máquina virtual recebe e envia os pacotes com o comando:

```c
ifconfig
```
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187135849956118599/image.png?ex=6595c96a&is=6583546a&hm=76adcc2bd357117cf98978daaf8abfc8ac0b980844739209d96020979d7c16f5&)

### 3. Depois de obter as interfaces, substituir no script python;

### 4. Correr o codigo python com privilégios elevados no container do atacante;
```c
chmod a+x mycode.py
python3 mycode.py
```
### 5. Fazer ping de uma container A para container B dos 3 containers sem ser do atacante ("Os seus ips obtêm-se pelo comando ifconfig");
### 6. Observar resultados:

![img](https://media.discordapp.net/attachments/1153998326274994216/1187135848949485698/image.png?ex=6595c96a&is=6583546a&hm=0091e659cc662a2b54998cfb091f810be55f6add100741543643021558a41913&=&format=webp&quality=lossless&width=1317&height=662)
![img](https://media.discordapp.net/attachments/1153998326274994216/1187135849243103252/image.png?ex=6595c96a&is=6583546a&hm=088b9c56775b67f34b06b3077fa36a6a56acb87eece4141597382e47f7fbae50&=&format=webp&quality=lossless&width=1440&height=303)

# Task 1.1B
> O objetivo foi conseguir filtrar os pacotes vindos para o pc do atacante alterando o parâmetro filtro do código python durante sniffing em "sniff()"

## Passos:
### 1. Conseguir filtrar pelo tipo de pacote ICMP que é dado pelo parâmetro "filter" no código python do exercício anterior:
```py
pkt = sniff(iface=interfaces, filter=f'src host {source_ip} and dst port {destination_port}', prn=print_pkt)
```
#### Resultados:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187156325101211749/image.png?ex=6595dc7c&is=6583677c&hm=3b4a0cce0d2e32adba974e31e907ff0a7f4de5648f5cb586aaa1435493347736&)
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187156325508071435/image.png?ex=6595dc7c&is=6583677c&hm=737d222b3129045771988d677569bc40e2a678d7e5219daa1fa40867cdd19a29&)

### 2. Conseguir filtrar pelo endereço ip de origem, pelo tipo de pacote e pelo número da porta dos pacotes que sofrem sniff, alterando a forma como se envia um pacote no computador onde envia com o seguinte script:

```py
from scapy.all import *


src_ip = '192.168.1.1'
dst_ip = '10.9.0.5'

# Define the destination port (Telnet port 23)
dst_port = 23


tcp_packet = IP(src=src_ip, dst=dst_ip) / TCP(sport=12345, dport=dst_port, flags='S')

send(tcp_packet)
```
#### Este script envia um pacote do tipo tcp para a porta designada 23, com o endereço ip origem definido para um dos containers das 3 que não é o transmissor nem atacante.

#### Resultados:
![img](https://media.discordapp.net/attachments/1153998326274994216/1187156325851996252/image.png?ex=6595dc7c&is=6583677c&hm=9130267af720cc69b8e13c8f6d66cf9679207e3043c265607b85064b8da36746&=&format=webp&quality=lossless&width=1440&height=574)
![img](https://media.discordapp.net/attachments/1153998326274994216/1187156326174953562/image.png?ex=6595dc7c&is=6583677c&hm=3a82f7a8fb56e9ff67c99bd2a396bb4e492afe53ab8403fdab915ca13204b19f&=&format=webp&quality=lossless&width=1440&height=574)

### 3. Conseguir filtrar pela subnet,quer seja pelo destino ou origem, com o seguinte filtro no parametro do filter usando na mesma o código python dado na task 1.1A:
```py
custom_filter= 'src net 192.168.1.0/24'
custom_filter2= 'dst net 128.230.0.0/16'
pkt = sniff(iface=interfaces, filter=custom_filter, prn=print_pkt)
```
#### Para testar usamos a mesma script no exercicio 2 desta task e alteramos o ip de acordo com os subnets.
#### Resultados:
![img](https://media.discordapp.net/attachments/1153998326274994216/1187156327009632367/image.png?ex=6595dc7c&is=6583677c&hm=15a7f507ca700a16ca0c3a24eb0d7cf806d08dd6d1faa778e158d0de0d266375&=&format=webp&quality=lossless&width=1440&height=547)
![img](https://media.discordapp.net/attachments/1153998326274994216/1187156327479386224/image.png?ex=6595dc7c&is=6583677c&hm=8089a53fc2b3452bcda29298a4afa4d87da7eee2397543907920ae5b6e48191a&=&format=webp&quality=lossless&width=1440&height=415)

# Task 1.3
> O objetivo desta task foi entender como forjar um pacote e enviar pelo código que serve de ferramenta para spoofing

## Passos
### 1. Usamos o seguinte código python para forjar um pacote e enviar pelo container atacante, spoofing:

```py 
from scapy.all import *
a = IP()
a.dst = '10.9.0.6'
b = ICMP()
p = a/b
send(p)
```

### 2. Depois abrimos Wireshark na interface onde foi enviado pacote, visto com ifconfig, ou na tab "any" onde ve as atividades de todas as interfaces de network.


### 3. Os resultados foram os seguintes:

![img](https://media.discordapp.net/attachments/1153998326274994216/1187159877802536981/image.png?ex=6595dfcb&is=65836acb&hm=ede7263082f6bf4bf7d49a792978f70b84fb1915ddae1af1b1f8e08217223b6e&=&format=webp&quality=lossless&width=1440&height=272)

# Task 1.3
> O objetivo desta task foi perceber como funciona o traceroute e implementar código em python usando scapy uma ferramenta parecida, e com ajuda do Wireshark.

## Passos

### 1. Forjar código python de modo que limite o número de routes o pacote pode passar e parar no momento quando o pacote chega ao destino:

```py 
from scapy.all import *
a = IP()
a.dst = '1.1.1.1'

for i in range(1,255):
    a.ttl=i
    b = ICMP()
    p = a/b
    response = sr1(p)
    if response:
        if a.dst==response[IP].src:
            print("Timeout at :",i,"\n")  
            break

```
### 2. Enviamos o pacote no container atacante e reparamos que o pacote chega quando passa por 15 routers:
![img](https://media.discordapp.net/attachments/1153998326274994216/1187170356113457212/image.png?ex=6595e98d&is=6583748d&hm=00a4851189900c826b54bc3014e51766e8f209f6ee44fe1b807b367b8d42b047&=&format=webp&quality=lossless&width=1214&height=662)

### 3. Verificamos no Wireshark e reparamos que existe um conjunto de 30 pedidos e respostas, ou seja 15 pedidos e paragens:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187170652801740901/image.png?ex=6595e9d4&is=658374d4&hm=e54bd53b49bb9b5ee68362b2ef9f8602c23c5f7e4c6b1401d1ad1f38c362a2a8&)
![img](https://media.discordapp.net/attachments/1153998326274994216/1187170356910366745/image.png?ex=6595e98d&is=6583748d&hm=c40ce9f3665076978cc921bef43eda9044cdca13c8003ae2253e49a45878172d&=&format=webp&quality=lossless&width=1440&height=417)

#### Conseguimos também ver que endereços ip é que tem as paragens/routers neste programa.

# Task 1.4
> Combinando os conhecimentos dos exercícios anteriores, o objetivo desta task é conseguir sniffar um ICMP request e enviar a resposta para a origem que fez "ping" logo que recebemos a request, assim conseguimos fazer com que a origem receba uma resposta mesmo o destino original esteja morto e não consiga responder ao ICMP request.

## Passos
### 1. No container do atacante , correr o seguinte código com priviégios elevados:
```py
#!/usr/bin/env python3
from scapy.all import *

filter_icmp_request = 'icmp and icmp[0]==8'
a = IP()
b = ICMP(type=0, code=0)

def spoof(pkt):
	pkt.show()
	src_ip = pkt[IP].src
	dst_ip = pkt[IP].dst
	a.src = dst_ip
	a.dst = src_ip
	p = a / b
	send(p)

interfaces = get_if_list()
sniff(iface=interfaces, filter=filter_icmp_request, prn=spoof)
```

#### O código acima filtra os pacotes por ICMP requests com o icmp[0]==8 sendo o 8 tipo de pacote ICMP.
#### A funcao spoof executa depois de ter capturado um pacote ICMP request e envia um pacote de resposta com destino a origem do pacote.

### 2. Fazer ping no container sem ser atacante

#### Resultados depois de ping 1.2.3.4
![img](https://media.discordapp.net/attachments/1153998326274994216/1187847409783541871/image.png?ex=6598601b&is=6585eb1b&hm=ea57cdf3d2ebd735f8813095dc44f4c2ce71d628c6b4659154f3268e154f70b0&=&format=webp&quality=lossless&width=873&height=457)
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187847409427021915/image.png?ex=6598601b&is=6585eb1b&hm=41cd5ddf65769931d1b85069c4e704774ae79daa1ea485069f818b23f80c780e&)

Como este host é inválido não conseguimos obter uma resposta por parte do servidor. Mas o nosso spoof ia conseguir fazer com que o packet resposta seja enviado ao host que fez ping.
Verificamos que o sniff capturou os pacotes request.

#### Resultados do ping a 10.9.0.99:
O resultado é o mesmo, pois esse ip não existe no LAN local e pacote não consegue chegar.

#### Resultado do ping a 8.8.8.8
Como é um ip válido, este consegue obter uma resposta do host 8.8.8.8, conseguimos também fazer sniff e enviar um pacote de tipo resposta:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187852251847462973/image.png?ex=6598649e&is=6585ef9e&hm=604f068dd1e34eb8220665817f62582864a40e7c580e02868e59823ece7deaa9&)

![img](https://media.discordapp.net/attachments/1153998326274994216/1187852252203974787/image.png?ex=6598649e&is=6585ef9e&hm=cebe912fc49be51274b5d0dd22b5858d90d0d624136fab866bf36fd691f022f4&=&format=webp&quality=lossless&width=625&height=337)
![img](https://media.discordapp.net/attachments/1153998326274994216/1187852252547911700/image.png?ex=6598649e&is=6585ef9e&hm=92c2df463cc8c0a982db716d99569a96bb4350cab2dbf3f1b16620307bae7968&=&format=webp&quality=lossless&width=625&height=337)

