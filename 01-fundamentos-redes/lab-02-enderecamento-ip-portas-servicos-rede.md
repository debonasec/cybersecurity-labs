# LAB 02 — ENDEREÇAMENTO IP, PORTAS E SERVIÇOS DE REDE
```text
│
├── Objetivo
├── Ambiente
├── Comandos utilizados
├── Resultados observados
├── Endereçamento IPv4 e sub-redes
├── IPv6
├── Portas e sockets
├── TCP e UDP
├── DNS, DHCP e NAT
├── Exercícios de endereçamento
├── Glossário de redes
├── Formação complementar
└── O que aprendi / Conclusão
```

## OBJETIVO
Compreender na prática conceitos fundamentais de endereçamento e comunicação em redes, incluindo IPv4, IPv6, sub-redes, portas, TCP, UDP, DNS, DHCP e NAT.

O laboratório também teve como objetivo praticar cálculos de sub-redes, interpretar configurações reais da máquina Kali Linux e relacionar endereços IP, interfaces, rotas, portas e protocolos observados no sistema.

## AMBIENTE
* Data da execução: 12/08/2026
* Sistema operacional: Kali Linux
* Ambiente: Máquina virtual - modo de rede NAT
* Interface de rede principal: `eth0`
* IPv4 da máquina: `10.0.2.15/24`
* Gateway IPv4 padrão: `10.0.2.2`
* IPv6: habilitado
* IPv6 link-local observado: `fe80::a00:27ff:fe64:4cda`
* Gateway IPv6 observado: `fe80::2`

## COMANDOS UTILIZADOS
```bash
ip addr
ip route
ip -6 addr
ip -6 route
ping -6 -c 5 ::1
ss -tulpn
ss -ltn
ss -lun
sudo ss -ltnp
```

Durante a análise de uma porta TCP específica também foi utilizado:

```bash
sudo ss -ltnp 'sport = :39351'
```

## RESULTADOS OBSERVADOS

### Endereço IPv4
A interface principal utilizada pela máquina foi `eth0`, com o endereço IPv4:

```text
10.0.2.15/24
```

O prefixo `/24` corresponde à máscara:
```text
255.255.255.0
```

Nesse endereço, os primeiros 24 bits representam o prefixo da rede e os demais bits ficam disponíveis para identificar hosts dentro da sub-rede.

### Gateway IPv4
O gateway padrão observado foi:

```text
10.0.2.2
```

A rota padrão é utilizada quando a máquina precisa enviar tráfego para um destino para o qual não existe uma rota mais específica.

De forma simplificada:

```text
Kali
10.0.2.15
    │
    ▼
Gateway
10.0.2.2
    │
    ▼
outras redes
```

### Endereçamento IPv6
O comando:

```bash
ip -6 addr
```

mostrou diferentes tipos de endereços IPv6 na máquina.

Entre eles foram observados:

```text
::1/128
```

utilizado como endereço de loopback IPv6.

Na interface `eth0` foram observados endereços ULA:

```text
fd17:625c:f037:2:991f:7b5e:464c:2c04/64
fd17:625c:f037:2:a00:27ff:fe64:4cda/64
```

Também foi observado o endereço link-local:

```text
fe80::a00:27ff:fe64:4cda/64
```

O endereço link-local é utilizado para comunicação no enlace local e não deve ser encaminhado por roteadores para outros enlaces.

### Rotas IPv6
O comando:

```bash
ip -6 route
```

apresentou as seguintes rotas relevantes, aqui exibidas de forma resumida:

```text
fd17:625c:f037:2::/64 dev eth0 proto ra
fe80::/64 dev eth0
default via fe80::2 dev eth0 proto ra
```

A rota:

```text
default via fe80::2
```

indica que `fe80::2` estava sendo utilizado como gateway IPv6 padrão.

A indicação:

```text
proto ra
```

está relacionada ao Router Advertisement, mecanismo utilizado por roteadores IPv6 para anunciar informações da rede.

### Teste do loopback IPv6
Foi executado:

```bash
ping -6 -c 5 ::1
```

Resultado observado:

```text
5 packets transmitted
5 received
0% packet loss
```

O teste confirmou o funcionamento do endereço de loopback IPv6 da própria máquina.

## ENDEREÇAMENTO IPv4 E SUB-REDES
Um endereço IPv4 possui 32 bits, divididos em quatro grupos de 8 bits chamados octetos.

Exemplo:

```text
192.168.1.50
```

Cada octeto pode assumir valores entre `0` e `255`.

A máscara de sub-rede determina quais bits pertencem à parte de rede e quais ficam disponíveis para os hosts.

Exemplo:

```text
IP:      192.168.1.50
Máscara: 255.255.255.0
CIDR:    /24
```

De forma simplificada:

```text
192.168.1 | 50
─────────   ──
   rede     host
```

### CIDR
A notação CIDR indica a quantidade de bits utilizados no prefixo da rede.

Exemplos:

```text
/24 → 255.255.255.0
/25 → 255.255.255.128
/26 → 255.255.255.192
/27 → 255.255.255.224
/28 → 255.255.255.240
```

Quanto maior o prefixo, menor é a quantidade de endereços disponíveis dentro daquela sub-rede.

### Rede, hosts e broadcast
Em uma sub-rede IPv4 tradicional existem endereços com funções diferentes.

Exemplo:

```text
192.168.1.0/24
```

Temos:

```text
Rede:          192.168.1.0
Primeiro host: 192.168.1.1
Último host:   192.168.1.254
Broadcast:     192.168.1.255
```

O endereço de rede identifica a própria sub-rede.

O endereço de broadcast é utilizado em situações de comunicação destinada aos hosts da mesma rede IPv4.

Para sub-redes IPv4 convencionais, a quantidade total de endereços pode ser calculada por:

```text
2^(32 - prefixo)
```

Tradicionalmente, dois endereços são descontados da quantidade de hosts utilizáveis: o endereço da rede e o endereço de broadcast.

Existem exceções, como prefixos `/31` e `/32`, que possuem usos específicos e não seguem essa regra da mesma maneira.

## IPv6
O IPv6 é a versão 6 do Internet Protocol e utiliza endereços de 128 bits escritos em hexadecimal.

Exemplo:

```text
2001:db8::1
```

O IPv6 permite omitir zeros à esquerda e comprimir uma sequência contínua de grupos formados por zeros utilizando `::`.

A abreviação `::` pode ser utilizada apenas uma vez em um mesmo endereço para que ele continue sendo interpretado de forma não ambígua.

No ambiente observado, os endereços IPv6 da interface `eth0` utilizavam prefixo `/64`.

Nesse caso, de forma simplificada:

```text
64 bits → prefixo da rede
64 bits → identificador da interface
```

### Tipos observados
**Loopback**

```text
::1
```

É utilizado para a máquina comunicar-se consigo mesma.

**Link-local**

Normalmente pertence à faixa:

```text
fe80::/10
```

É utilizado para comunicação dentro do enlace local.

**ULA — Unique Local Address**

Pertence à faixa:

```text
fc00::/7
```

Endereços iniciados por `fd` são frequentemente utilizados nesse contexto.

Eles são destinados ao uso local e não equivalem a endereços IPv6 públicos da Internet.

### Dual stack
A máquina analisada possuía IPv4 e IPv6 funcionando simultaneamente.

Essa configuração é conhecida como:

```text
Dual Stack
```

Representação simplificada:

```text
Kali Linux
   │
   ├── IPv4 → 10.0.2.15
   │
   └── IPv6 → fd17:... / fe80:...
```

## PORTAS E SOCKETS
O endereço IP ajuda a identificar uma interface ou destino na rede, enquanto a porta identifica um endpoint de transporte normalmente associado a uma aplicação ou serviço.

Uma forma simples de visualizar é:

```text
IP    → em qual máquina/interface?
Porta → qual endpoint de aplicação?
```

Exemplo:

```text
192.168.1.100:443
```

Onde:

```text
192.168.1.100 → endereço IP
443           → porta
```

Portas também são específicas ao protocolo de transporte.

Por isso:

```text
53/tcp
```

e:

```text
53/udp
```

não representam o mesmo endpoint de transporte.

### Porta de origem e porta de destino
Uma comunicação pode ser representada como:

```text
10.0.2.15:51832 → 93.184.216.34:443
```

Nesse exemplo:

```text
IP de origem:       10.0.2.15
Porta de origem:    51832

IP de destino:      93.184.216.34
Porta de destino:   443
```

A porta `51832` pode representar uma porta efêmera escolhida temporariamente pelo sistema operacional do cliente.

A porta `443` é convencionalmente utilizada por serviços HTTPS, embora uma aplicação possa ser configurada para utilizar outras portas.

### Estado LISTEN
Durante o laboratório, o comando:

```bash
ss -ltn
```

apresentou inicialmente um socket semelhante a:

```text
LISTEN ... 127.0.0.1:39351
```

Isso indicava que existia um processo aguardando novas conexões TCP na porta `39351`, associado ao endereço de loopback.

Como o endereço utilizado era:

```text
127.0.0.1
```

o serviço estava restrito à própria máquina.

Posteriormente, ao consultar novamente a porta `39351`, ela não apareceu mais no resultado.

Isso mostrou na prática que uma porta não precisa permanecer permanentemente em uso: um processo pode encerrar, reiniciar ou deixar de utilizar aquele socket.

### Endereço de escuta
Foram estudadas três situações:

```text
127.0.0.1:8000
```

Indica um serviço associado ao loopback e acessível localmente.

```text
10.0.2.15:8000
```

Indica um serviço associado especificamente à interface que possui o endereço `10.0.2.15`.

```text
0.0.0.0:8000
```

Indica um serviço TCP vinculado a todas as interfaces IPv4 disponíveis.

Isso não significa automaticamente que o serviço esteja acessível por qualquer máquina externa, pois firewall, roteamento e outras configurações também podem limitar o acesso.

## TCP E UDP
TCP e UDP funcionam na camada de transporte do modelo TCP/IP.

### TCP
O TCP é orientado à conexão e fornece às aplicações um fluxo de bytes confiável e ordenado.

Antes da troca normal de dados, uma conexão TCP normalmente passa pelo processo conhecido como 3-way handshake:

```text
CLIENTE                         SERVIDOR

   SYN  -------------------------->

        <------------------- SYN-ACK

   ACK  -------------------------->

        conexão estabelecida
```

De maneira simplificada:

```text
SYN
"Quero iniciar uma conexão."

SYN-ACK
"Recebi e também estou pronto."

ACK
"Confirmado."
```

### UDP
O UDP é um protocolo de transporte sem estabelecimento prévio de conexão e, por isso, não utiliza o 3-way handshake do TCP. Ele não fornece, por si só, as mesmas garantias de entrega, retransmissão e ordenação oferecidas pelo TCP.

No Lab 01, foi observado DNS utilizando:

```text
UDP/53
```

enquanto a comunicação HTTPS analisada utilizou:

```text
TCP/443
```

Isso não significa que DNS utilize somente UDP. O protocolo também pode utilizar TCP na porta 53 em determinadas situações.

Da mesma forma, HTTPS não deve ser entendido como obrigatoriamente limitado a TCP, já que diferentes versões e tecnologias podem utilizar outros protocolos de transporte.

## DNS, DHCP E NAT

### DNS
O DNS é utilizado, entre outras funções, para resolver nomes de domínio em endereços IP.

De maneira simplificada:

```text
example.com
     │
     ▼
    DNS
     │
     ▼
endereço IP
```

Pergunta mental:

```text
"Qual é o endereço IP deste nome?"
```

### DHCP
O DHCP permite fornecer automaticamente configurações de rede aos dispositivos.

Entre as informações que podem ser fornecidas estão:

```text
Endereço IP
Máscara/prefixo
Gateway
Servidor DNS
Tempo de concessão
```

Pergunta mental:

```text
"Qual configuração de rede devo utilizar?"
```

### NAT
NAT significa Network Address Translation.

É um mecanismo utilizado para traduzir endereços durante a comunicação entre redes.

Em um cenário comum:

```text
Máquina
IP privado
    │
    ▼
   NAT
    │
    ▼
outra rede
```

No ambiente virtual utilizado neste laboratório, a máquina Kali possui o endereço privado:

```text
10.0.2.15
```

A máquina virtual estava configurada em modo NAT. Nesse cenário,
o mecanismo de virtualização realiza a tradução necessária para
permitir a comunicação entre a rede privada da VM e redes externas.

### Resumo
```text
DHCP
"Qual configuração de rede devo utilizar?"

DNS
"Qual IP corresponde a este nome?"

Gateway
"Por onde envio o tráfego para outra rede?"

NAT
"Como os endereços serão traduzidos entre redes?"
```

## EXERCÍCIOS DE ENDEREÇAMENTO
Durante o estudo foram resolvidos exercícios com diferentes prefixos CIDR.

| Endereço         | CIDR  | Máscara           | Rede             | Primeiro host    | Último host      | Broadcast        | Total | Utilizáveis |
| ---------------- | ----- | ----------------- | ---------------- | ---------------- | ---------------- | ---------------- | ----: | ----------: |
| `192.168.1.50`   | `/24` | `255.255.255.0`   | `192.168.1.0`    | `192.168.1.1`    | `192.168.1.254`  | `192.168.1.255`  |   256 |         254 |
| `10.10.10.77`    | `/24` | `255.255.255.0`   | `10.10.10.0`     | `10.10.10.1`     | `10.10.10.254`   | `10.10.10.255`   |   256 |         254 |
| `192.168.10.70`  | `/25` | `255.255.255.128` | `192.168.10.0`   | `192.168.10.1`   | `192.168.10.126` | `192.168.10.127` |   128 |         126 |
| `192.168.10.150` | `/26` | `255.255.255.192` | `192.168.10.128` | `192.168.10.129` | `192.168.10.190` | `192.168.10.191` |    64 |          62 |
| `192.168.10.173` | `/27` | `255.255.255.224` | `192.168.10.160` | `192.168.10.161` | `192.168.10.190` | `192.168.10.191` |    32 |          30 |
| `192.168.50.186` | `/28` | `255.255.255.240` | `192.168.50.176` | `192.168.50.177` | `192.168.50.190` | `192.168.50.191` |    16 |          14 |
| `10.20.30.214`   | `/27` | `255.255.255.224` | `10.20.30.192`   | `10.20.30.193`   | `10.20.30.222`   | `10.20.30.223`   |    32 |          30 |

Os exercícios foram utilizados para praticar:

```text
máscara
prefixo CIDR
tamanho do bloco
endereço de rede
primeiro host
último host
broadcast
quantidade total de endereços
quantidade de hosts utilizáveis
```

## GLOSSÁRIO DE REDES

### 01. Bit
Menor unidade de informação binária, podendo assumir o valor `0` ou `1`.

### 02. Octeto
Grupo formado por 8 bits. Um endereço IPv4 possui quatro octetos.

### 03. IPv4
Versão 4 do Internet Protocol, que utiliza endereços de 32 bits.

### 04. IPv6
Versão 6 do Internet Protocol, que utiliza endereços de 128 bits.

### 05. Máscara de sub-rede
Valor de 32 bits utilizado no IPv4 para determinar quais bits identificam a rede e quais identificam os hosts.

### 06. CIDR
Método de endereçamento e roteamento sem classes que utiliza prefixos de tamanho variável, normalmente representados pela notação /n.

### 07. Prefixo de rede
Parte inicial de um endereço IP que identifica a rede ou sub-rede. Seu comprimento pode ser representado por /n, como /24.

### 08. Sub-rede
Divisão lógica de uma rede maior em redes menores.

### 09. Endereço de rede
Endereço utilizado para identificar uma sub-rede IPv4.

### 10. Host
Sistema final conectado a uma rede, que pode possuir uma ou mais interfaces e endereços IP.

### 11. Broadcast
Forma de comunicação IPv4 utilizada para enviar tráfego a todos os hosts de uma determinada rede local.

### 12. Endereço privado
No IPv4, endereço reservado para utilização em redes privadas e que não é roteado diretamente na Internet pública.

### 13. Endereço público
Endereço IP destinado à comunicação e ao roteamento na Internet pública.

### 14. Gateway padrão
Dispositivo ou próximo salto utilizado normalmente para encaminhar tráfego destinado a outras redes.

### 15. Rota
Informação utilizada pelo sistema para determinar por onde encaminhar tráfego até determinado destino.

### 16. Rota padrão
Rota utilizada quando não existe uma rota mais específica para o destino.

### 17. Interface de rede
Ponto de conexão físico ou virtual utilizado por um sistema para comunicar-se através de uma rede.

### 18. Loopback
Interface ou endereço utilizado pela máquina para comunicar-se consigo mesma.

### 19. Link-local
Utilizado para comunicação no enlace local e não deve ser encaminhado por roteadores para outros enlaces.

### 20. ULA
Unique Local Address. Faixa de endereços IPv6 destinada ao uso em redes locais.

### 21. Dual stack
Configuração na qual IPv4 e IPv6 funcionam simultaneamente no mesmo sistema ou rede.

### 22. Router Advertisement (RA)
Mensagem IPv6 utilizada por roteadores para anunciar sua presença e fornecer informações de configuração da rede.

### 23. Endereço MAC
Identificador associado a uma interface de rede e utilizado principalmente na comunicação na camada de enlace.

### 24. Porta
Número utilizado pelo protocolo de transporte para identificar endpoints associados às aplicações.

### 25. Porta de origem
Porta do endpoint que está enviando aquele segmento ou datagrama.

### 26. Porta de destino
Porta do endpoint que deve receber aquele segmento ou datagrama.

### 27. Porta efêmera
Porta atribuída dinamicamente pelo sistema operacional, comumente utilizada pelo lado cliente de uma comunicação.

### 28. Socket
Endpoint de comunicação gerenciado pelo sistema operacional que, em TCP/IP, pode estar associado a protocolo, endereço e porta.

### 29. LISTEN
Estado TCP no qual um socket está aguardando novas conexões.

### 30. TCP
Protocolo de transporte orientado à conexão que fornece às aplicações um fluxo de bytes confiável e ordenado.

### 31. UDP
Protocolo de transporte sem estabelecimento prévio de conexão que não fornece, por si só, as mesmas garantias de entrega, retransmissão e ordenação oferecidas pelo TCP.

### 32. SYN
Flag TCP utilizada durante o estabelecimento de uma conexão e na sincronização inicial dos números de sequência.

### 33. SYN-ACK
Combinação de flags TCP utilizada para reconhecer um SYN e também iniciar a sincronização no sentido oposto durante o 3-way handshake.

### 34. ACK
Flag TCP utilizada para indicar confirmação do recebimento de segmentos.

### 35. 3-way handshake
Processo utilizado pelo TCP para estabelecer uma conexão através da troca de SYN, SYN-ACK e ACK.

### 36. DNS
Domain Name System. Sistema utilizado, entre outras funções, para resolver nomes de domínio em endereços IP.

### 37. DHCP
Dynamic Host Configuration Protocol. Protocolo utilizado para fornecer automaticamente configurações de rede aos dispositivos.

### 38. NAT
Network Address Translation. Mecanismo utilizado para traduzir endereços durante a comunicação entre redes.

### 39. Endereço IPv6 não especificado
O endereço `::` representa a ausência de um endereço IPv6 específico. É um endereço especial e não deve ser atribuído a uma interface.

### 40. Roteamento
Processo de determinar caminhos ou próximos saltos utilizados para alcançar outras redes. Essas informações são utilizadas no encaminhamento dos pacotes.

## FORMAÇÃO COMPLEMENTAR
Este laboratório foi desenvolvido como parte dos estudos de fundamentos de redes e cibersegurança, em paralelo ao curso **Introduction to Cybersecurity — Cisco Networking Academy**.

A formação complementar reforçou conceitos relacionados a:

* funcionamento das redes;
* endereçamento de dispositivos;
* comunicação entre sistemas;
* protocolos;
* funcionamento da Internet;
* princípios básicos de segurança da informação.

O laboratório foi utilizado para transformar esses conceitos em exercícios práticos e documentação técnica.

## O QUE APRENDI / CONCLUSÃO
Este foi um dos laboratórios em que tive mais dificuldade até agora. Diferente de alguns conteúdos anteriores, vários conceitos de endereçamento e comunicação em redes exigiram mais tempo, exercícios e revisões para que eu começasse a entender como eles se relacionam na prática.

Pratiquei cálculos de sub-redes IPv4 com diferentes prefixos CIDR, trabalhando com endereço de rede, máscara, primeiro e último host, broadcast e quantidade de endereços disponíveis. Apesar de ter conseguido resolver os exercícios propostos, ainda quero aprofundar subnetting para ganhar mais segurança e conseguir realizar esses cálculos com maior naturalidade.

Também analisei a configuração IPv6 da minha máquina Kali Linux, identificando endereços de loopback, link-local e ULA, além de observar Router Advertisement, rota padrão IPv6 e o funcionamento de IPv4 e IPv6 em dual stack. IPv6 ainda é um dos assuntos que considero necessário revisar e praticar mais.

Na camada de transporte, consegui compreender melhor a relação entre endereços IP, portas e aplicações. Analisei sockets TCP em estado `LISTEN`, portas de origem e destino, portas efêmeras e revisei as diferenças entre TCP e UDP, incluindo o processo de 3-way handshake com SYN, SYN-ACK e ACK.

Também organizei melhor as funções de DNS, DHCP, gateway e NAT. Antes, alguns desses conceitos ainda se misturavam para mim; com os exercícios e exemplos práticos, comecei a entender melhor o papel de cada um dentro de uma comunicação em rede.

Durante este laboratório utilizei o ChatGPT como ferramenta de apoio aos estudos, principalmente para tirar dúvidas, revisar minhas respostas, identificar erros de entendimento e explicar novamente conceitos que eu não havia compreendido de primeira. Esse suporte foi importante para conseguir avançar no conteúdo, mas não considero que tenha assimilado 100% de todos os assuntos abordados.

Por isso, este laboratório representa mais um registro do meu processo de aprendizado do que um domínio completo do tema. Ainda preciso aprofundar principalmente subnetting, IPv6 e alguns detalhes de roteamento e transporte, retomando esses conceitos nos próximos estudos e laboratórios.

Mesmo com as dificuldades, a prática me ajudou a conectar assuntos que antes pareciam separados e a construir uma visão mais clara de como endereçamento, roteamento, portas, protocolos e serviços trabalham juntos durante uma comunicação em rede.






