# Simulação de Ataques e Detecção com Suricata IDS (Kali Linux + Ubuntu)

Este repositório documenta um home lab completo de segurança de rede, no qual múltiplos tipos de ataques foram simulados utilizando Kali Linux enquanto um servidor Ubuntu monitorado pelo Suricata IDS realizava a detecção das atividades maliciosas.

O objetivo é demonstrar, de forma prática, tanto o lado ofensivo quanto o defensivo da segurança, simulando etapas reais de um ataque e analisando como um sistema de detecção de intrusão identifica essas atividades.

## 🎯 Objetivo do Lab

Simular diferentes tipos de ataques de rede

Monitorar o tráfego utilizando Suricata IDS

Identificar alertas gerados pelo sistema de detecção

Analisar logs gerados durante os ataques

Demonstrar um fluxo completo de detecção e resposta a incidentes

## 🧪 Ambiente Utilizado
Sistema Atacante

Sistema operacional: Kali Linux

IP: 192.168.56.129

Ferramentas utilizadas:

Nmap

Hydra

Gobuster

Hping3

Sistema Alvo / Monitoramento

Sistema operacional: Ubuntu Server

IP: 192.168.56.128

IDS: Suricata

Firewall: iptables

Interface de rede: ens33

Rede do laboratório

Ambiente virtual com rede privada:
```
Kali Linux (Attacker)
192.168.56.129
        │
        │ simulated attacks
        ▼
Ubuntu Server (Target)
192.168.56.128
        │
        ▼
Suricata IDS monitoring network traffic
```

## 🛠 Ferramentas Utilizadas
Sistema de detecção

Suricata IDS – detecção de intrusão em rede

Ferramentas ofensivas

Nmap – varredura de rede e identificação de serviços

Hydra – ataque de força bruta em autenticação

Gobuster – enumeração de diretórios web

Hping3 – geração de pacotes TCP e simulação de SYN flood

## 🔍 Metodologia

Durante o laboratório, diferentes fases de um ataque foram simuladas.

Cada etapa gerou tráfego de rede que foi monitorado e analisado pelo Suricata IDS.

### 1️⃣ Reconhecimento de rede

O primeiro passo de um atacante geralmente é identificar portas abertas e serviços ativos.

Comando executado no Kali Linux:
```
nmap -sS -T4 -A 192.168.56.128
```
Esse comando executa:

SYN scan (detecção de portas abertas)

identificação de serviços

detecção de sistema operacional

execução de scripts de reconhecimento

<img width="1021" height="508" alt="image" src="https://github.com/user-attachments/assets/1637c5eb-93e1-4a9c-9bbe-d3b8b57da5ed" />

Resultado observado:

22/tcp open ssh
80/tcp open http

Alertas detectados pelo Suricata:

ET SCAN Possible Nmap User-Agent Observed

<img width="1294" height="801" alt="image" src="https://github.com/user-attachments/assets/be631374-449a-4be9-a973-7be34f5b55f1" />

Isso indica atividade de network scanning.

### 2️⃣ Verificação de vulnerabilidades

Após identificar os serviços, foi executado um scan de vulnerabilidades utilizando scripts NSE.
```
nmap --script vuln 192.168.56.128
```
Esse comando executa scripts que testam vulnerabilidades conhecidas.

Exemplos testados:

XSS

CSRF

vulnerabilidades HTTP

Resultado observado:

No vulnerabilities found

<img width="584" height="312" alt="image" src="https://github.com/user-attachments/assets/f8ea5991-1f7a-4dc8-b22b-6fbd44734547" />

Mesmo sem vulnerabilidades detectadas, o tráfego foi registrado pelo IDS como atividade de scanning.

### 3️⃣ Enumeração de diretórios web

O atacante então tentou identificar diretórios e arquivos ocultos no servidor web.

Ferramenta utilizada: Gobuster
```
gobuster dir -u http://192.168.56.128 -w /usr/share/wordlists/dirb/common.txt
```
Resultado:

/.htaccess
/.htpasswd
/index.html
/server-status

Esses resultados indicam diretórios existentes no servidor.

Esse tipo de enumeração é frequentemente utilizado para descobrir:

painéis administrativos

endpoints ocultos

arquivos de configuração

<img width="758" height="485" alt="image" src="https://github.com/user-attachments/assets/ce770421-3250-4d02-9e5a-95ae7d6fd6b7" />

### 4️⃣ Ataque de força bruta SSH

O atacante tentou obter acesso ao servidor via SSH utilizando força bruta.

Ferramenta utilizada: Hydra
```
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.128
```
Esse ataque gera múltiplas tentativas de autenticação.

<img width="1281" height="207" alt="image" src="https://github.com/user-attachments/assets/13542e95-efc2-4b90-aee9-0e30a5688824" />

Logs do sistema Ubuntu indicaram:

Failed password for root from 192.168.56.129

Esse comportamento é característico de ataques de força bruta.

<img width="816" height="800" alt="image" src="https://github.com/user-attachments/assets/cfe929b1-7334-4369-a916-e8013d21cb9e" />

### 5️⃣ Simulação de ataque DoS (SYN Flood)

Para simular um ataque de negação de serviço, foi utilizado o Hping3.
```
sudo hping3 -S -p 80 --flood 192.168.56.128
```
Esse comando envia um grande volume de pacotes TCP SYN para o servidor.

<img width="631" height="159" alt="image" src="https://github.com/user-attachments/assets/cb133023-8cde-4c80-8bda-3b257ceb962f" />

Consequências observadas:

grande volume de conexões TCP

geração massiva de logs no Suricata

saturação do armazenamento de logs

Erro registrado pelo Suricata:

No space left on device

<img width="1284" height="819" alt="image" src="https://github.com/user-attachments/assets/ebca040b-7765-4ad0-a748-4b33f2d341ed" />

Isso ocorreu porque o ataque gerou um volume extremamente alto de eventos no arquivo:
```
/var/log/suricata/eve.json
```

## 🛡 Detecção de Atividades pelo Suricata

Durante os ataques, o Suricata gerou múltiplos alertas.

Exemplo de alerta:
```
ET SCAN Possible Nmap User-Agent Observed
```
Outro alerta observado:
```
Possible Kali Linux hostname in DHCP Request Packet
```

<img width="1313" height="48" alt="image" src="https://github.com/user-attachments/assets/4be934db-8311-43ad-aa07-5db14bd009af" />

Esse alerta indica que o IDS detectou a presença de um host identificado como Kali Linux na rede.

## 🚨 Resposta ao Incidente

Após identificar o IP atacante, foi simulada uma resposta defensiva utilizando firewall.

Regra aplicada:
```
sudo iptables -A INPUT -s 192.168.56.129 -j DROP
```
Essa regra bloqueia todo o tráfego proveniente do atacante.

<img width="1155" height="82" alt="image" src="https://github.com/user-attachments/assets/f6c8746b-399d-4c4c-a197-b57f1a790e7b" />

Após a aplicação da regra, novas tentativas de ataque foram testadas.

Comando executado no Kali:
```
hydra -l root -P rockyou.txt ssh://192.168.56.128
```
Resultado observado:
```
Timeout connecting to 192.168.56.128
```

<img width="1264" height="216" alt="image" src="https://github.com/user-attachments/assets/17107f6b-9fa2-4329-9c84-0f18501b7125" />

Isso confirma que o firewall bloqueou com sucesso o tráfego do atacante.

Fluxo de resposta:
```
Ataque detectado
↓
Identificação do IP atacante
↓
Aplicação de regra no firewall
↓
Bloqueio do tráfego malicioso
```

## 📊 Resultados Observados

Durante o laboratório foi possível observar:

detecção de scans de rede

identificação de enumeração web

múltiplas tentativas de autenticação SSH

tráfego anômalo gerado por SYN flood

geração massiva de alertas no IDS

saturação de logs causada por tráfego volumétrico

mitigação do ataque utilizando firewall

Esse cenário simula de forma realista o funcionamento de um ambiente de monitoramento de segurança de rede.

# 📚 Conclusão

Este laboratório demonstra na prática como diferentes fases de um ataque podem ser detectadas por um sistema de detecção de intrusão em rede.

A simulação inclui desde reconhecimento inicial até tentativas de comprometimento e ataques de negação de serviço, oferecendo uma visão realista do funcionamento de um ambiente de monitoramento de segurança.

O uso combinado de ferramentas ofensivas e defensivas permite compreender melhor como ataques são conduzidos e como podem ser identificados e mitigados em ambientes reais.
