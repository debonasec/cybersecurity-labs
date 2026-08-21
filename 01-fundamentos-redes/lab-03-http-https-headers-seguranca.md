# LAB 03 — HTTP, HTTPS E ANÁLISE DE REQUISIÇÕES

```text
│
├── Objetivo
├── Ambiente
├── Comandos utilizados
├── HTTP — requisição e resposta
├── Métodos e status HTTP
├── Headers
├── HTTPS, TLS e certificados
├── Cadeia de confiança
├── Proxy, VPN e firewall
├── Autenticação e autorização
├── Captura de requisição com Burp Suite
├── Análise da captura
├── Análise de riscos
├── Formação complementar
└── O que aprendi / Conclusão
```

## OBJETIVO

Compreender melhor o funcionamento de HTTP e HTTPS, analisando na prática uma comunicação entre cliente e servidor.

O laboratório teve como objetivo identificar elementos de uma requisição e de uma resposta HTTP, incluindo método, caminho, host, código de status e headers.

Também foram estudados conceitos relacionados a TLS, certificados digitais, cadeia de confiança, proxy, VPN, firewall, autenticação e autorização.

Como prática principal, uma requisição HTTP foi capturada utilizando o Burp Suite e analisada com foco nas informações presentes e em possíveis aspectos relevantes para segurança.

## AMBIENTE

* Data da execução: 21/08/2026
* Sistema operacional: Kali Linux
* Ambiente: máquina virtual
* Ferramentas utilizadas:

  * `curl`
  * OpenSSL
  * Burp Suite Community
  * navegador configurado para utilizar o proxy do Burp Suite
* Site utilizado para os testes: `https://example.com`

O domínio `example.com` foi utilizado por ser apropriado para testes e exemplos, sem necessidade de interação com sistemas de terceiros que não tenham sido destinados para esse tipo de atividade.

## COMANDOS UTILIZADOS

Inicialmente foram utilizados comandos com `curl` para observar uma comunicação HTTPS.

```bash
curl -I https://example.com
```

O parâmetro `-I` permite solicitar apenas os headers da resposta, utilizando normalmente o método HTTP `HEAD`.

Também foi utilizado:

```bash
curl -v https://example.com -o /dev/null
```

O parâmetro `-v` habilita a saída detalhada da comunicação.

O parâmetro:

```text
-o /dev/null
```

faz com que o corpo da resposta seja descartado, permitindo concentrar a análise na conexão, TLS, requisição e headers da resposta.

Para analisar diretamente o certificado apresentado pelo servidor, foram utilizados:

```bash
openssl s_client -connect example.com:443 -servername example.com
```

e:

```bash
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -subject -issuer -dates -ext subjectAltName
```

Para visualizar informações relacionadas à cadeia de certificados também foi utilizado:

```bash
openssl s_client -connect example.com:443 -servername example.com -showcerts
```

## HTTP — REQUISIÇÃO E RESPOSTA

HTTP permite a comunicação entre clientes e servidores por meio de requisições e respostas.

De forma simplificada:

```text
CLIENTE                              SERVIDOR

        requisição HTTP
        --------------------------->

                       resposta HTTP
        <---------------------------
```

Uma requisição pode conter informações como:

```text
método
caminho
host
headers
corpo, quando aplicável
```

Uma resposta pode conter:

```text
versão HTTP
código de status
headers
corpo
```

Durante o teste realizado com `curl`, foi observada uma requisição semelhante a:

```http
GET / HTTP/2
Host: example.com
User-Agent: curl/...
Accept: */*
```

Nesse caso:

```text
GET          → método
/            → caminho solicitado
example.com  → host
HTTP/2       → versão HTTP utilizada nessa conexão
```

Na saída detalhada do `curl`:

```text
>
```

indicava informações enviadas pelo cliente.

Já:

```text
<
```

indicava informações recebidas do servidor.

## MÉTODOS E STATUS HTTP

### Métodos HTTP

O método indica a ação que o cliente deseja realizar sobre determinado recurso.

Alguns métodos estudados foram:

```text
GET
→ solicitar um recurso

POST
→ enviar dados para processamento

PUT
→ criar ou substituir uma representação de um recurso

DELETE
→ solicitar a remoção de um recurso

HEAD
→ solicitar os headers associados ao recurso sem retornar
   o corpo da resposta
```

Durante o laboratório foram observados principalmente os métodos `GET` e `HEAD`.

### Códigos de status

Os códigos HTTP podem ser agrupados por categoria:

```text
1xx → informação
2xx → sucesso
3xx → redirecionamento
4xx → problemas relacionados à requisição/cliente
5xx → problemas no servidor
```

Alguns códigos estudados:

```text
200 OK
→ requisição processada com sucesso

301 Moved Permanently
→ redirecionamento permanente

302 Found
→ redirecionamento

400 Bad Request
→ requisição inválida

401 Unauthorized
→ autenticação ausente ou inválida

403 Forbidden
→ acesso não autorizado para aquele recurso

404 Not Found
→ recurso não encontrado

500 Internal Server Error
→ erro interno do servidor
```

Durante os testes foi observado:

```text
HTTP/2 200
```

indicando sucesso no processamento da requisição.

## HEADERS

Headers transportam informações adicionais sobre a requisição ou resposta.

Na requisição foram estudados exemplos como:

```http
Host: example.com
User-Agent: ...
Accept: ...
Accept-Encoding: ...
Connection: ...
```

### Host

O header `Host` identifica o host para o qual a requisição é destinada.

Exemplo:

```http
Host: example.com
```

### User-Agent

O `User-Agent` fornece informações sobre o cliente utilizado para realizar a requisição.

Dependendo do cliente, ele pode revelar informações como:

```text
navegador
versão aproximada
sistema operacional
plataforma
```

A presença dessas informações não representa automaticamente uma vulnerabilidade.

### Accept

O header `Accept` informa quais tipos de conteúdo o cliente consegue ou prefere receber.

Exemplos:

```text
text/html
application/json
image/webp
*/*
```

O valor:

```text
*/*
```

indica que outros tipos de conteúdo também podem ser aceitos.

### Content-Type

Na resposta foi observado:

```http
Content-Type: text/html
```

Esse header informa que o conteúdo retornado deve ser interpretado como HTML.

### Server

Também foi observado:

```http
Server: cloudflare
```

Esse header fornece uma informação sobre a infraestrutura que participou do atendimento da requisição.

Essa informação, isoladamente, não representa uma vulnerabilidade.

### Allow

Foi observado:

```http
Allow: GET, HEAD
```

O header informa que, naquele contexto, os métodos `GET` e `HEAD` são suportados ou permitidos pelo servidor.

## HEADERS DE SEGURANÇA

Durante o estudo também foram analisados alguns headers relacionados a mecanismos de proteção utilizados por navegadores.

### Strict-Transport-Security

Também conhecido como HSTS.

Exemplo:

```http
Strict-Transport-Security: max-age=31536000
```

Esse header orienta o navegador a utilizar HTTPS para aquele site durante o período estabelecido pela política.

### X-Content-Type-Options

Exemplo:

```http
X-Content-Type-Options: nosniff
```

O valor `nosniff` orienta o navegador a não tentar interpretar o conteúdo como um tipo diferente daquele declarado pelo servidor em `Content-Type`.

### Content-Security-Policy

Também conhecido como CSP.

A Content Security Policy permite definir regras sobre as origens das quais determinados conteúdos podem ser carregados, como scripts, imagens e outros recursos.

A ausência de um determinado header de segurança, isoladamente, não comprova que exista uma vulnerabilidade.

É necessário analisar o contexto e o possível impacto antes de classificar uma condição como vulnerável.

## HTTPS, TLS E CERTIFICADOS

HTTPS pode ser entendido, de forma simplificada, como HTTP protegido através de TLS.

```text
HTTP
 +
TLS
 ↓
HTTPS
```

Durante a execução de:

```bash
curl -v https://example.com -o /dev/null
```

foi observada uma conexão utilizando:

```text
TLS 1.3
```

Também foi negociado:

```text
h2
```

através de ALPN, indicando o uso de HTTP/2 nessa comunicação entre `curl` e o servidor.

A sequência observada pode ser representada de forma simplificada como:

```text
resolução DNS
      ↓
conexão com a porta 443
      ↓
handshake TLS
      ↓
certificado apresentado e validado
      ↓
canal TLS estabelecido
      ↓
requisição HTTP
      ↓
resposta HTTP
```

### Proteções associadas ao TLS

Foram estudados três conceitos principais:

```text
Confidencialidade
→ proteção contra leitura indevida do conteúdo durante o transporte

Integridade
→ permite detectar alterações indevidas na comunicação

Autenticação
→ permite verificar a identidade apresentada pelo servidor
```

## CERTIFICADOS DIGITAIS

O certificado apresentado por `example.com` foi analisado diretamente utilizando OpenSSL.

Foram observadas informações como:

```text
subject=CN=example.com

issuer=C=US, O=SSL Corporation,
CN=Cloudflare TLS Issuing ECC CA 3
```

### Subject

O `subject` identifica o sujeito ao qual o certificado foi emitido.

No certificado analisado:

```text
CN=example.com
```

### Issuer

O `issuer` indica quem emitiu ou assinou o certificado.

Foi observado:

```text
CN=Cloudflare TLS Issuing ECC CA 3
```

O emissor apresentado não precisa ser necessariamente a CA raiz final da cadeia de confiança.

### Período de validade

Também foram observadas as datas:

```text
notBefore=Jul 29 22:10:08 2026 GMT
notAfter=Oct 27 22:17:21 2026 GMT
```

`notBefore` indica a partir de quando o certificado é considerado válido.

`notAfter` indica até quando ele é considerado válido.

### Subject Alternative Name — SAN

Foi observado:

```text
DNS:example.com
DNS:*.example.com
```

O SAN informa os nomes para os quais o certificado pode ser considerado válido.

O valor:

```text
*.example.com
```

é um wildcard utilizado para representar subdomínios compatíveis daquele nível, como:

```text
www.example.com
teste.example.com
```

O domínio:

```text
example.com
```

aparece separadamente porque o wildcard `*.example.com` não cobre automaticamente o domínio raiz.

Um certificado possuir validade temporal e uma cadeia confiável não é suficiente se o hostname acessado não corresponder aos nomes válidos presentes no certificado.

## CADEIA DE CONFIANÇA

Durante a análise com OpenSSL foi observado:

```text
Certificate chain
0 s:CN=example.com
  i:C=US, O=SSL Corporation,
    CN=Cloudflare TLS Issuing ECC CA 3
```

Nesse trecho:

```text
s:
→ subject

i:
→ issuer
```

De forma simplificada, uma cadeia pode ser representada como:

```text
Certificado do site
        │
        ▼
CA intermediária
        │
        ▼
CA raiz confiável
```

Uma CA intermediária pode assinar certificados utilizados pelos servidores.

Essa CA, por sua vez, deve permitir a construção de uma cadeia de confiança até uma autoridade raiz considerada confiável pelo sistema.

Durante o teste com `curl` também foi observado o uso da loja de certificados do sistema:

```text
/etc/ssl/certs/ca-certificates.crt
```

Se a cadeia não puder ser validada até uma raiz confiável, o cliente pode apresentar um erro ou alerta de certificado.

Um certificado válido também não significa que todo site seja confiável ou que não possa possuir conteúdo malicioso.

HTTPS protege a comunicação e ajuda a verificar a identidade associada ao domínio, mas não garante a legitimidade de todas as ações realizadas pelo responsável pelo site.

## PROXY, VPN E FIREWALL

Os três conceitos possuem funções diferentes.

### Proxy

Um proxy atua como intermediário entre cliente e destino.

```text
CLIENTE
   │
   ▼
 PROXY
   │
   ▼
SERVIDOR
```

No laboratório, o Burp Suite foi utilizado dessa maneira:

```text
Navegador
    │
    ▼
Burp Proxy
    │
    ▼
example.com
```

Isso permite visualizar a comunicação HTTP que passa pelo proxy.

### VPN

Uma VPN cria um túnel entre o dispositivo e outro ponto da rede.

```text
Kali
  │
  │ túnel VPN
  ▼
Rede remota
```

Esse tipo de recurso é utilizado, por exemplo, para permitir acesso autorizado a redes de laboratórios remotos.

A existência de uma VPN não elimina regras de firewall, roteamento ou outros controles presentes na rede.

### Firewall

Um firewall aplica regras para decidir qual tráfego deve ser permitido ou bloqueado.

Exemplo conceitual:

```text
tráfego
   │
   ▼
firewall
   │
   ├── permitir
   │
   └── bloquear
```

Proxy, VPN e firewall não possuem a mesma função e podem existir simultaneamente dentro de uma mesma comunicação.

## AUTENTICAÇÃO E AUTORIZAÇÃO

Autenticação e autorização representam conceitos diferentes.

### Autenticação

Responde à pergunta:

```text
"Quem é você?"
```

Exemplos:

```text
usuário e senha
MFA
token
certificado
biometria
```

### Autorização

Responde à pergunta:

```text
"O que você pode fazer?"
```

Exemplo:

```text
usuário comum
→ acessar funções comuns

administrador
→ acessar funções administrativas
```

Um usuário pode estar autenticado e, ainda assim, não possuir autorização para determinada ação.

De forma simplificada:

```text
Autenticação
→ identidade

Autorização
→ permissões
```

## CAPTURA DE REQUISIÇÃO COM BURP SUITE

Como prática principal do laboratório, o navegador foi configurado para encaminhar sua comunicação através do proxy do Burp Suite.

O `Intercept` foi mantido desligado e as requisições foram observadas através de:

```text
Proxy
↓
HTTP history
```

Foi acessado:

```text
https://example.com
```

Uma das requisições observadas foi:

```http
GET / HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html,...
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

A resposta correspondente apresentou informações como:

```http
HTTP/2 200 OK
Content-Type: text/html
Server: cloudflare
Allow: GET, HEAD
Cf-Cache-Status: HIT
```

Na captura, o Burp apresentou a requisição como HTTP/1.1 e a resposta do servidor como HTTP/2.

Como o Burp atua como intermediário, a conexão entre navegador e Burp e a conexão entre Burp e servidor podem utilizar versões diferentes do protocolo HTTP.

```text
Navegador
    │
    ▼
Burp
    │
    ▼
Servidor
```

## ANÁLISE DA CAPTURA

A requisição analisada apresentou:

```text
Método:          GET
Caminho:         /
Host:            example.com
Cliente:         identificado através do User-Agent
```

O header `Accept` informou quais tipos de conteúdo o navegador estava preparado para receber.

A resposta apresentou:

```text
Status:          200 OK
Resultado:       sucesso
Content-Type:    text/html
Server:          cloudflare
Allow:           GET, HEAD
```

O código:

```text
200 OK
```

indicou que a requisição foi processada com sucesso.

O header:

```text
Content-Type: text/html
```

informou que o corpo da resposta possuía conteúdo HTML.

O header:

```text
Server: cloudflare
```

forneceu uma informação relacionada à infraestrutura utilizada para responder à requisição.

Isso não representa automaticamente uma vulnerabilidade.

## ANÁLISE DE RISCOS

Na requisição e na resposta analisadas, foram observadas informações como `User-Agent` e `Server`.

O `User-Agent` pode revelar informações sobre o navegador, sistema operacional e plataforma utilizados pelo cliente.

O header `Server` pode revelar informações relacionadas à infraestrutura que participou do atendimento da requisição.

Essas informações podem ser relevantes durante uma análise, mas isoladamente não são suficientes para caracterizar uma vulnerabilidade.

Não foram observados na captura utilizada para documentação dados sensíveis como:

```text
Cookie de sessão
Authorization
senha
token de acesso
API key
```

Headers como:

```http
Authorization: Bearer <token>
```

devem ser tratados como informações sensíveis, pois um token válido pode funcionar como uma credencial e permitir ações de acordo com as permissões associadas a ele.

Esse tipo de informação não deve ser publicado em documentação pública.

Também foram estudados headers de segurança, como:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
```

A ausência de um desses headers em uma resposta não foi considerada, isoladamente, prova de uma vulnerabilidade.

É necessário avaliar:

```text
informação observada
        ↓
contexto
        ↓
possível impacto
        ↓
validação
        ↓
classificação do risco
```

Na captura analisada, **não foi identificada uma vulnerabilidade comprovada**.

## FORMAÇÃO COMPLEMENTAR

Este laboratório foi desenvolvido como parte dos estudos de fundamentos de redes e segurança web, em paralelo ao curso **Introduction to Cybersecurity — Cisco Networking Academy**.

Os estudos anteriores de redes também foram importantes para compreender melhor o fluxo observado neste laboratório, principalmente conceitos como:

* DNS;
* endereços IP;
* portas;
* TCP;
* cliente e servidor;
* protocolos;
* comunicação em rede.

## O QUE APRENDI / CONCLUSÃO

Neste laboratório consegui relacionar conceitos de redes estudados anteriormente com o funcionamento de uma aplicação web.

Utilizando `curl`, acompanhei partes da comunicação realizada ao acessar um site HTTPS e consegui identificar a negociação TLS, a utilização de HTTP/2, o método HTTP enviado e o código de status recebido.

Também utilizei OpenSSL para analisar diretamente informações do certificado apresentado pelo servidor. Com isso, consegui compreender melhor conceitos como `subject`, `issuer`, período de validade, Subject Alternative Name e cadeia de confiança.

Alguns desses conceitos, principalmente certificados e cadeia de confiança, ainda exigem mais revisão para que eu consiga compreendê-los com maior profundidade e sem depender tanto de consultas durante a análise.

A diferença entre proxy, VPN e firewall também ficou mais clara. Entendi que um proxy atua como intermediário, uma VPN cria um túnel para outro ponto ou rede e um firewall utiliza regras para permitir ou bloquear tráfego.

Também consegui separar melhor autenticação de autorização: autenticação está relacionada à comprovação de identidade, enquanto autorização determina quais recursos ou ações aquela identidade pode acessar.

A parte mais prática do laboratório foi a utilização do Burp Suite para capturar uma requisição real. Analisei método, caminho, host, headers e código de status da resposta.

Durante a análise também comecei a desenvolver um cuidado que considero importante para os próximos estudos: não tratar toda informação encontrada como uma vulnerabilidade.

Headers como `User-Agent` e `Server` podem revelar informações relevantes, mas a presença deles, isoladamente, não significa que exista uma falha de segurança. Da mesma forma, a ausência de um determinado header de proteção precisa ser analisada dentro do contexto e do impacto possível.

O ChatGPT foi utilizado como ferramenta de apoio durante todo o laboratório para revisar minhas respostas, explicar novamente conceitos que não ficaram claros de primeira e corrigir interpretações equivocadas.

Ainda preciso aprofundar principalmente TLS, certificados, cadeia de confiança e análise de headers de segurança. Mesmo assim, este laboratório me ajudou a compreender melhor como uma requisição HTTPS pode ser analisada desde o estabelecimento da conexão até a interpretação dos dados observados no Burp Suite.

