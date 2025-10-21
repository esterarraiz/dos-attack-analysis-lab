# Estudo de Caso: Análise de Ataques de Negação de Serviço (DoS)

**Autor:** Ester Arraiz de Matos  
**Data de Execução:** Outubro de 2025  
**GitHub:** esterarraiz  
**LinkedIn:** Ester Arraiz de Matos  

---

> ⚠️ **AVISO LEGAL E ÉTICO IMPORTANTE**  
> Este projeto foi conduzido com fins estritamente educacionais em um ambiente de laboratório virtual, isolado e controlado.  
> O objetivo é analisar os efeitos de diferentes tipos de ataques de Negação de Serviço (DoS) para compreender seus mecanismos e desenvolver melhores estratégias de defesa.  
> A execução de ataques DoS contra sistemas de terceiros é ilegal, antiética e causa danos reais.  
> Não utilize as técnicas ou ferramentas aqui demonstradas fora de um ambiente de teste de sua propriedade.  
> A autora não se responsabiliza por qualquer uso indevido deste material.

---

## 1. Resumo do Projeto

Este repositório documenta a simulação de três tipos distintos de ataques de Negação de Serviço (DoS) contra a máquina vulnerável **Metasploitable 2**.  
Utilizando a ferramenta **Pentmenu** no Kali Linux, foram executados ataques que visam diferentes camadas do modelo OSI (Camada 7, 4 e 3) para observar e analisar seus impactos específicos no sistema alvo.

O objetivo principal é demonstrar como cada ataque explora uma fraqueza diferente e como um defensor pode usar ferramentas como `top`, `netstat` e `ping` para diagnosticar o tipo de ataque em andamento.

### Evidências
Todas as imagens e evidências do processo estão organizadas na pasta a seguir. Clique nos links para navegar diretamente para as capturas de tela de cada ataque.  
Pasta Principal: [images/](images/)  
Ataque ICMP Echo Flood: [images/ICMP Echo Flood/](images/ICMP%20Echo%20Flood/)  
Ataque Slowloris: [images/Slowloris/](images/Slowloris/)  
Ataque TCP SYN Flood: [images/TCP SYN Flood/](images/TCP%20SYN%20Flood/)


---

## 2. Ambiente do Laboratório

- **Máquina Atacante:** Kali Linux (IP: 192.168.49.4)  
- **Máquina Alvo:** Metasploitable 2 (IP: 192.168.49.5)  
- **Software de Virtualização:** VirtualBox  
- **Configuração de Rede:** Rede Interna (Host-Only)  
- **Ferramenta de Ataque:** Pentmenu  

---

## 3. Metodologia e Execução dos Ataques

Após a configuração do ambiente, executei três cenários de ataque, cada um focado em uma camada diferente do modelo de rede.

---

### Cenário 1: Ataque de Camada 7 - Slowloris

#### Explicação do Ataque
O **Slowloris** é um ataque de negação de serviço *low-and-slow* que visa a camada de aplicação (Camada 7).  
Em vez de inundar o alvo com tráfego, ele abre múltiplas conexões com o servidor web e as mantém abertas pelo maior tempo possível, enviando cabeçalhos HTTP parciais de forma lenta e contínua.  
Isso esgota o pool de conexões disponíveis no servidor, impedindo que usuários legítimos consigam se conectar.

#### Execução com Pentmenu
1. Naveguei até o diretório do pentmenu e executei `./pentmenu`.  
2. Selecionei a opção **2) DOS**.  
3. Selecionei a opção **9) Slowloris**.  
4. Configurei os parâmetros do ataque:
   - **Target:** 192.168.49.5  
   - **Port:** 80 (servidor Apache)  
   - **Number of connections:** 2000 (máximo)  
   - **Interval:** r (aleatório)

#### Impacto e Análise
- **Serviço Web:** A aplicação web (DVWA) tornou-se completamente inacessível, com a página em um estado de carregamento infinito.  
- **Conexões:** O comando `netstat -ntp | grep :80` na máquina vítima revelou um número massivo de conexões em estado `ESTABLISHED`.  
- **Recursos do Servidor:** O comando `top` mostrou um aumento drástico no número de processos `apache2`.  
- **Latência:** A latência da rede (medida com `ping`) aumentou consideravelmente, com picos ainda maiores após o término do ataque.

---

### Cenário 2: Ataque de Camada 4 - TCP SYN Flood

#### Explicação do Ataque
O **TCP SYN Flood** explora o funcionamento do three-way handshake (SYN, SYN-ACK, ACK) da camada de transporte (Camada 4).  
O atacante envia um grande volume de pacotes TCP SYN para o servidor, muitas vezes com IPs de origem falsificados.  
O servidor responde a cada um com um SYN-ACK e aloca recursos para esperar pelo ACK final, que nunca chega.  
Isso preenche a fila de conexões pendentes do servidor, impedindo novas conexões legítimas.

#### Execução com Pentmenu
1. No menu DoS do Pentmenu, selecionei a opção **3) TCP SYN Flood**.  
2. Configurei os parâmetros:
   - **Target:** 192.168.49.5  
   - **Port:** 80  
   - **Source IP:** r (aleatório)

#### Impacto e Análise
- **Conexões:** O comando `netstat` na vítima mostrou um grande número de conexões na porta 80 presas no estado `SYN_RECV`.  
- **Recursos do Servidor:** O `top` mostrou um aumento significativo no uso da CPU (categoria `si` - *software interrupts*).  
- **Serviço Web:** O serviço web se tornou inacessível.

---

### Cenário 3: Ataque de Camada 3 - ICMP Echo Flood

#### Explicação do Ataque
O **ICMP Echo Flood** é um ataque que visa a camada de rede (Camada 3).  
O objetivo é saturar a largura de banda da rede ou esgotar os recursos da CPU, enviando um volume massivo de pacotes ICMP Echo Request (pings).  
O sistema alvo é forçado a processar e responder (ICMP Echo Reply) a cada pacote, consumindo seus recursos.

#### Execução com Pentmenu
1. No menu DoS, selecionei a opção **1) ICMP Echo Flood**.  
2. Configurei os parâmetros:
   - **Target:** 192.168.49.5  
   - **Source IP:** r (aleatório)

#### Impacto e Análise
- **Latência e Conectividade:** Degradação severa da conectividade da rede, com aumento drástico na latência e perda de pacotes.  
- **Análise de Serviços:** O `netstat` na porta 80 não mostrou anomalias, e o `top` não mostrou aumento nos processos do Apache.  
  Isso ocorre porque o ataque visa a infraestrutura de rede do alvo, não uma aplicação específica.

---

## 4. Recomendações de Mitigação

Cada tipo de ataque requer uma estratégia de defesa diferente:

### Contra Slowloris (Camada 7)
- **Ajuste de Servidores Web:** Aumentar o número máximo de clientes e diminuir os *timeouts* de conexão.  
- **Reverse Proxies e Load Balancers:** Utilizar Nginx ou HAProxy para absorver conexões lentas antes que cheguem ao servidor principal.  
- **Web Application Firewalls (WAFs):** Configurar para detectar e bloquear padrões de tráfego *slow-rate*.

### Contra TCP SYN Flood (Camada 4)
- **SYN Cookies:** O servidor só aloca recursos após receber o ACK final.  
- **Firewalls e IPS:** Utilizar dispositivos com proteção integrada contra SYN Floods.  
- **Ajuste do Kernel:** Aumentar o tamanho da fila de conexões pendentes (*backlog*).

### Contra ICMP Flood (Camada 3)
- **Regras de Firewall:** Bloquear ou limitar a taxa de tráfego ICMP vindo de fontes não confiáveis.  
- **Provedor de Internet (ISP):** Solicitar bloqueio do tráfego malicioso em ataques de grande volume.

---

## 5. Conclusão

Este estudo prático demonstrou que ataques de Negação de Serviço são multifacetados, com cada tipo deixando uma "impressão digital" diferente no sistema alvo.  
Para um profissional de segurança (especialmente do *Blue Team*), saber identificar esses sintomas — seja através do estado das conexões em `netstat` ou do consumo de recursos em `top` — é crucial para diagnosticar rapidamente o tipo de ataque e aplicar a mitigação correta.

---
