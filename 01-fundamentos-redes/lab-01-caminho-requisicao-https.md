# LAB 01 — CAMINHO DE UMA REQUISIÇÃO HTTPS
```text
│
├── Objetivo
├── Ambiente
├── Comandos utilizados
├── Diagrama da requisição
├── Resultados observados
├── OSI x TCP/IP
├── Aplicação
├── Transporte
├── Internet
├── Acesso à rede
├── Cliente e servidor
├── Encapsulamento e desencapsulamento
├── Formação complementar
└── O que aprendi / Conclusão
```

## OBJETIVO
Entender o caminho percorrido por uma requisição HTTPS, desde a resolução do nome do domínio até a resposta do servidor, relacionando o processo às camadas do modelo TCP/IP.


## AMBIENTE
- Data da execução: 10/08/2026
- Sistema operacional: Kali Linux
- Interface de rede: `eth0`
- IPv4 da máquina: `10.0.2.15/24`
- Gateway padrão: `10.0.2.2`
- Servidor DNS observado: `168.227.139.16`
- Site utilizado no teste: https://example.com


## COMANDOS UTILIZADOS
```bash
ip addr
ip route
dig example.com
curl -I https://example.com
```


## DIAGRAMA DA REQUISIÇÃO
```text
CLIENTE
Kali Linux
10.0.2.15
eth0
    │
    │
    │ 1. Resolver example.com
    ▼
DNS
168.227.139.16
UDP / 53
    │
    │ retorna:
    │ 172.66.147.243
    │ 104.20.23.154
    ▼
IP DO DESTINO
    │
    │ destino fora da rede local
    ▼
GATEWAY
10.0.2.2
    │
    ▼
INTERNET
    │
    ▼
INFRAESTRUTURA DO DESTINO
    │
    │
    │ 2. TCP / 443
    ▼
CONEXÃO TCP
    │
    │
    │ 3. TLS
    ▼
CANAL PROTEGIDO
    │
    │
    │ 4. HTTP
    ▼
REQUISIÇÃO HTTP
Método: HEAD
Destino: example.com
    │
    ▼
RESPOSTA
HTTP/2 200
```

> **Nota:** O diagrama apresenta as etapas de forma simplificada.
> Como o servidor DNS observado também está fora da rede local,
> a consulta DNS é encaminhada pelo gateway.


## RESULTADOS OBSERVADOS

### Interface e endereço IP
O comando `ip addr` mostrou que a interface utilizada durante o
laboratório foi `eth0`, com o endereço:

```text
10.0.2.15/24
```

### Gateway padrão
O comando `ip route` retornou:

```text
default via 10.0.2.2 dev eth0
```

O endereço `10.0.2.2` é, portanto, o gateway padrão utilizado pela
máquina para alcançar redes externas.

### Resolução DNS
A consulta:

```bash
dig example.com
```

utilizou o servidor DNS `168.227.139.16` e retornou, durante o
laboratório, os endereços IPv4:

```text
172.66.147.243
104.20.23.154
```

A consulta DNS observada utilizou UDP na porta 53.

### Resposta HTTP
O comando:

```bash
curl -I https://example.com
```

retornou:

```text
HTTP/2 200
```

O código de status `200 (OK)` indica que a requisição foi bem-sucedida.


## OSI x TCP/IP
O modelo OSI possui sete camadas e é muito utilizado para compreender e diagnosticar comunicações de rede. O modelo TCP/IP agrupa essas funções em quatro camadas e representa de forma mais próxima a arquitetura utilizada na Internet.

| Modelo OSI                       | Modelo TCP/IP | Exemplos        |
| -------------------------------- | ------------- | --------------- |
| Aplicação, Apresentação e Sessão | Aplicação     | HTTP, DNS, TLS  |
| Transporte                       | Transporte    | TCP, UDP        |
| Rede                             | Internet      | IPv4, IPv6      |
| Enlace e Física                  | Acesso à rede | Ethernet, Wi-Fi |

```text   
┌────────────────────────────────────┐
│ APLICAÇÃO                          │
│ DNS, HTTP, TLS*                    │
│                                    │
│ "O que quero fazer?"               │
├────────────────────────────────────┤
│ TRANSPORTE                         │
│ TCP / UDP                          │
│                                    │
│"Como os dados serão transportados?"│
├────────────────────────────────────┤
│ INTERNET                           │
│ IPv4 / IPv6                        │
│                                    │
│ "Para qual endereço?"              │
├────────────────────────────────────┤
│ ACESSO À REDE                      │
│ Ethernet / eth0                    │
│                                    │
│ "Como envio pela rede local?"      │
└────────────────────────────────────┘
```
> **Nota sobre TLS:** neste modelo TCP/IP simplificado, o TLS está
> representado junto à camada de aplicação. O modelo TCP/IP não possui
> uma camada específica equivalente às camadas de sessão e apresentação
> do modelo OSI.


## APLICAÇÃO (DNS, HTTP e TLS)
É a camada mais próxima do que o usuário e os programas querem fazer. O DNS resolve `example.com` para um ou mais endereços IP. O HTTP sobre TLS forma a comunicação que normalmente chamamos de HTTPS.


## TRANSPORTE (TCP, UDP)
É responsável pelo transporte de dados entre aplicações. Neste laboratório, a consulta DNS foi realizada usando UDP na porta 53, enquanto a comunicação HTTPS utilizou TCP na porta 443. O TCP fornece às aplicações um fluxo de dados confiável e ordenado.


## INTERNET (IP)
É responsável pelo endereçamento e encaminhamento dos pacotes entre redes. Neste laboratório, o Kali possui o endereço `10.0.2.15/24` e utiliza o gateway `10.0.2.2` para alcançar destinos fora de sua rede local.


## ACESSO À REDE (Ethernet/eth0)
É a parte que coloca a comunicação efetivamente na rede local e permite que sua máquina entregue os dados ao próximo dispositivo, como o gateway. Neste laboratório, a interface utilizada foi `eth0`.


## CLIENTE E SERVIDOR
Neste laboratório, o `curl` atua como cliente, iniciando uma requisição
para `example.com`. A infraestrutura que recebe a requisição e envia a
resposta atua como servidor.

O cliente inicia a comunicação e o servidor processa a solicitação e
retorna uma resposta.


## ENCAPSULAMENTO E DESENCAPSULAMENTO
Durante o envio dos dados, cada camada adiciona informações necessárias
para realizar sua função.

De forma simplificada:

`Dados da aplicação → TCP → IP → Ethernet → Rede`

Esse processo é chamado de **encapsulamento**.

Quando os dados chegam ao destino, ocorre o processo inverso:

`Rede → Ethernet → IP → TCP → Dados da aplicação`

Esse processo é chamado de **desencapsulamento**.


## FORMAÇÃO COMPLEMENTAR
Este laboratório foi desenvolvido em paralelo aos estudos do curso
**Introduction to Cybersecurity — Cisco Networking Academy**.

A formação reforçou conceitos abordados neste laboratório, como:

- fundamentos de redes;
- funcionamento da Internet;
- endereçamento e comunicação entre dispositivos;
- ameaças, vulnerabilidades e riscos;
- princípios básicos de segurança da informação.

O laboratório foi utilizado para transformar parte desses conceitos
teóricos em uma prática documentada.


## O QUE APRENDI/CONCLUSÃO
Neste laboratório analisei o caminho de uma requisição HTTPS a partir
de uma máquina Kali Linux e entendi que acessar um site envolve diferentes
protocolos e etapas, e não apenas uma conexão direta entre o navegador
e o servidor.

Identifiquei o endereço IP da máquina, o gateway padrão e o servidor
DNS utilizado. Observei que o DNS é responsável por resolver um nome de domínio para um ou mais endereços IP e que o gateway encaminha tráfego destinado a outras redes.

Também relacionei a comunicação às camadas do modelo TCP/IP. Na camada
de aplicação estão protocolos como DNS e HTTP; na camada de transporte,
TCP e UDP; na camada de Internet, o IP; e na camada de acesso à rede,
tecnologias como Ethernet.

Na comunicação HTTPS estudada, o cliente resolve o domínio, estabelece
uma conexão TCP, utiliza TLS para proteger a comunicação e então troca
requisições e respostas HTTP com o servidor.

Por fim, compreendi o conceito de encapsulamento: cada camada acrescenta
informações necessárias para transportar os dados até o destino, onde o
processo é realizado de forma inversa.








