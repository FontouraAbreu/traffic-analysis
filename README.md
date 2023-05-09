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

`192.150.11 - pool-98-114-205-102.phlapa.fios`

`98.114.205 - wrasse.adobe.com`

***

## 2. O que você consegue descobrir sobre a máquina atacante (ex.: onde ela está localizada)? (1pt)

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

Portanto, o ataque levou cerca de 16 segundos.

***
