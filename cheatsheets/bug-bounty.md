# Bug Bounty

## Bug Bounty: Segredos dos Pros - Golden Window Strategy

> **Data:** 2025-11-10\
> **Status:** 🔥 OPERACIONAL\
> **Tags:** #bugbounty #strategy #automation #goldmine #consulting

***

### 🎯 MINDSET: O JOGO REAL

#### A Verdade Que Ninguém Fala

**99% dos hunters estão fazendo errado:**

* Gastam 3 semanas em recon de programas saturados
* Competem com 10k+ hunters pelos mesmos targets
* Ganham $200 após 60h de trabalho ($3/hora = salário mínimo)

**Os 1% que faturam 6 dígitos:**

* Focam em **janelas de ouro** (golden window: 2-4 semanas)
* Agem em **HORAS** quando novos alvos surgem
* Transformam bugs em **contratos de consultoria**
* Têm **acesso prioritário** via relacionamento direto com security teams

#### Equação do Sucesso

```
VELOCIDADE + ALVOS NOVOS + NETWORKING = $$$$$$
```

***

### 💰 A ESTRATÉGIA: GOLDEN WINDOW

#### O Que São "Alvos Novos"?

1. **Programas recém-lançados** (< 4 semanas)
2. **Empresas que fizeram aquisições** (assets adquiridos)
3. **Novos subdomínios/assets** adicionados a programas existentes
4. **Produtos/features novos** em empresas com bug bounty

#### Por Que Funciona?

| Fator                    | Impacto                         |
| ------------------------ | ------------------------------- |
| **Competição baixa**     | Poucos hunters perceberam ainda |
| **Vulns não exploradas** | Ninguém teve tempo de testar    |
| **Empresa motivada**     | Quer construir boa reputação    |
| **Pagamento rápido**     | Damage control mode ativado     |
| **Múltiplas vulns**      | Surface de ataque não hardened  |

***

### 🛠️ ARSENAL: FERRAMENTAS DOS PROS

#### 1. MONITORAMENTO AUTOMATIZADO (CRÍTICO)

**Monitorar Novos Programas**

**Plataformas com API:**

```bash
# HackerOne API
https://api.hackerone.com/v1/hackers/programs

# Bugcrowd (via scraping)
# Intigriti (via RSS/scraping)
```

**Ferramenta Custom (Python):**

```python
# bounty-monitor.py
# Checa APIs a cada 1h, notifica no Telegram/Discord
# Filtra por: scope, bounty range, launch date
```

**Ferramentas Prontas:**

* **BBScope** - https://github.com/sw33tLie/bbscope
* **Bounty Targets Data** - https://github.com/arkadiyt/bounty-targets-data
* **Chaos** (Project Discovery) - https://chaos.projectdiscovery.io

**Setup de Notificação:**

```bash
# Telegram Bot
# Discord Webhook
# Slack incoming webhook
```

***

**Monitorar Aquisições Corporativas**

**Fontes RSS/Scraping:**

* TechCrunch acquisitions feed
* Crunchbase news
* Yahoo Finance M\&A news
* SEC filings (empresas públicas)

**Automação:**

```bash
# RSS to Telegram/Discord
# Keywords: "acquired", "acquisition", "merger"
# Filtra empresas que TÊM bug bounty program
```

**Ferramenta Custom:**

```python
# acquisition-tracker.py
# Scrape RSS feeds
# Cross-reference com lista de empresas com BB programs
# Alert instantâneo
```

***

**Monitorar Novos Subdomínios**

**Ferramentas Essenciais:**

1.  **Sublert** - https://github.com/yassineaboukir/sublert

    ```bash
    # Monitora Certificate Transparency logs
    # Notificação real-time de novos subdomain
    python sublert.py -u target.com
    ```
2.  **crt.sh Monitor**

    ```bash
    # Script custom pra query crt.sh API
    # Diff com lista anterior
    # Alert no Telegram
    ```
3.  **Amass Track**

    ```bash
    # Amass modo tracking
    amass track -d target.com
    ```
4.  **Subfinder + notify**

    ```bash
    # Cron job a cada 6h
    subfinder -d target.com | notify -bulk
    ```

**Stack Completa de Monitoring:**

```bash
# Combo mortal
subfinder + amass + crt.sh + sublert
-> diff com baseline
-> notify via Telegram/Discord
```

***

#### 2. RECON RÁPIDO (FIRST 48H CRITICAL)

**Objetivo:** Low-hanging fruits em velocidade máxima

**Subdomain Enumeration (Speed Mode)**

```bash
# Passive (RÁPIDO)
subfinder -d target.com -silent | httpx -silent

# Active (se tiver tempo)
amass enum -passive -d target.com

# DNS bruteforce (se scope permitir)
puredns bruteforce wordlist.txt target.com
```

**Tech Stack Identification**

```bash
# Wappalyzer CLI
wappalyzer target.com

# WhatWeb
whatweb -a 3 target.com

# httpx tech detection
httpx -td -u target.com
```

**Quick Vulnerability Scan**

```bash
# Nuclei (templates specific)
nuclei -u target.com -t cves/ -t exposures/ -t misconfigurations/

# Nmap quick scan (se infra em scope)
nmap -sV -sC -oN scan.txt target.com

# Dirsearch (common paths)
dirsearch -u https://target.com -e php,asp,aspx,jsp
```

**Low-Hanging Fruits Priority**

1. **IDOR** (Insecure Direct Object Reference)
2. **Broken Authentication/Authorization**
3. **Exposed APIs** sem auth
4. **Subdomain takeover**
5. **Sensitive data exposure** (config files, backups, .git)
6. **CORS misconfiguration**
7. **Open redirects** (chain pra impact)

***

#### 3. AUTOMAÇÃO TOTAL (SCALING)

**Pipeline Automatizada:**

```bash
# 1. Descoberta
subfinder + amass + crt.sh
    ↓
# 2. Alive check
httpx (status, title, tech)
    ↓
# 3. Vulnerability scan
nuclei (templates relevantes)
    ↓
# 4. Manual verification
Browser + Burp Suite
    ↓
# 5. Report
Template profissional
```

**Script Mestre (bash/python):**

```python
#!/usr/bin/env python3
# auto-hunt.py

# Input: novo programa/asset
# Output: relatório com vulnerabilidades potenciais

1. subdomain_enum()
2. alive_check()
3. tech_detect()
4. vuln_scan()
5. generate_report()
6. notify_telegram()
```

**Ferramentas de Automação Pro:**

*   **Axiom** - https://github.com/pry0cc/axiom

    ```bash
    # Distributed recon na cloud (AWS/DO/GCP)
    # Escala horizontal massiva
    axiom-scan subdomains.txt -m nuclei -o results/
    ```
*   **Interlace** - https://github.com/codingo/Interlace

    ```bash
    # Threading de comandos
    interlace -tL targets.txt -c "nuclei -u _target_"
    ```
*   **Turbo Intruder** (Burp extension)

    ```python
    # High-speed fuzzing
    # Custom Python scripts
    ```

***

### 📊 WORKFLOW DOS PROS

#### Semana 1-2: Golden Window

```
DIA 1 (First 24h - CRÍTICO)
├── Alert de novo programa
├── 2h: Recon rápido (subdomains, tech stack)
├── 4h: Testa low-hanging fruits
├── Evening: Reporta primeiros bugs
└── Setup monitoring contínuo

DIA 2-3
├── Deep dive em funcionalidades críticas
├── Auth flows
├── Payment flows
├── File upload
└── API endpoints

DIA 4-7
├── Testa casos de edge
├── Chaining de vulns (low -> high impact)
├── Business logic flaws
└── Reporta findings

SEMANA 2
├── Follow-up nos reports
├── Email de consultoria (se bugs confirmados)
├── Networking com security team
└── Move pro próximo target novo
```

***

### 💼 DE BUG PRA CONSULTORIA (\$$\$$)

#### Email Template (Após Bug Confirmado)

```markdown
Subject: Security Assessment Proposal - [Company Name]

Hi [Security Team Contact],

Thank you for confirming the [vulnerability type] I reported 
(Report #[number]). 

Given my understanding of your infrastructure from this finding, 
I'd like to propose a comprehensive security assessment to help 
you get ahead of other researchers who will inevitably discover 
additional issues.

**Scope:**
- Full application security review
- API security testing
- Authentication/Authorization audit
- Business logic vulnerability assessment

**Deliverables:**
- Detailed technical report
- Prioritized remediation roadmap
- Secure coding recommendations
- Executive summary for stakeholders

**Timeline:** [X] weeks
**Investment:** $[Y]k fixed fee

I'm available to discuss this further at your convenience.

Best regards,
[Your Name]
[LinkedIn Profile]
[HackerOne/Bugcrowd Profile]
```

#### Follow-up Strategy

**Fase 1: First Contact**

* Report bug profissional (POC, impacto, remediação)
* Espera confirmação

**Fase 2: Upsell**

* Email de consultoria (template acima)
* Destaca conhecimento prévio da infra

**Fase 3: Delivery**

* Assessment completo
* Report profissional
* Vai além do escopo (mostra valor)

**Fase 4: Long-term**

* Mantém contato trimestral
* Oferece suporte em novas aquisições
* Vira "go-to researcher" deles

***

### 🔥 SEGREDOS QUE NINGUÉM CONTA

#### 1. **Private Programs > Public**

**Como conseguir convites:**

* Reputation alta em plataformas
* Reports de qualidade (não quantidade)
* Networking em conferências (DEF CON, Black Hat)
* LinkedIn: conecta com security teams
* Triage comments profissionais (ajuda outros hunters)

#### 2. **Timing de Reports**

**Envie reports:**

* ✅ Segunda-feira manhã (semana começa, atenção máxima)
* ✅ Terça/Quarta (ritmo normal)
* ❌ Sexta tarde (weekend = delay)
* ❌ Feriados (óbvio)

#### 3. **Severidade = Narrativa**

**Não é só a vuln, é como você conta:**

❌ **Ruim:** "IDOR no endpoint /api/user/{id}"

✅ **Bom:** "IDOR permitindo acesso não autorizado a dados sensíveis de 100k+ usuários, incluindo SSN, endereços e histórico de transações financeiras. Exploração trivial sem rate limiting."

**Elementos de report HIGH PAYOUT:**

1. **Impacto de negócio claro** (PII, financial loss, reputation)
2. **Proof of Concept detalhado** (screenshots, videos, curl commands)
3. **Steps to reproduce** (script quando possível)
4. **Suggested remediation** (código exemplo)
5. **Related vulnerabilities** (mostra profundidade)

#### 4. **Chaining = Critical**

**Single vuln = Medium**\
**Chain de vulns = Critical**

Exemplos:

* Open redirect + CSRF = Account takeover
* CORS misconfiguration + XSS = Data exfiltration
* IDOR + Race condition = Privilege escalation

#### 5. **Burp Extensions dos Pros**

```
Must-have:
├── Autorize (auth testing)
├── Turbo Intruder (fast fuzzing)
├── Param Miner (hidden params)
├── InQL (GraphQL testing)
├── JWT Editor (token manipulation)
├── Upload Scanner (file upload vulns)
└── Retire.js (outdated libs)
```

#### 6. **Wordlists Customizadas**

**Não usa wordlists genéricas:**

```bash
# Cria wordlist específica pro target
# Baseada em:
- Tech stack (Spring Boot? endpoints comuns)
- Industry (fintech, healthcare, etc.)
- Common naming patterns da empresa
- Leaked source code (GitHub dorking)

# Ferramentas:
- CeWL (wordlist from website)
- Commonspeak2 (specific to tech)
- Assetnote wordlists
```

#### 7. **GitHub Dorking (Antes do Recon)**

```bash
# Antes de começar, vê o que vazou
# GitHub dorks para [target]:
- "target.com" password
- "target.com" api_key
- "target.com" secret
- "target.com" token
- "target.com" credentials
- org:target-company filename:.env

# Ferramentas:
- truffleHog (secrets scanning)
- GitRob
- Gitrob
```

#### 8. **Traffic Proxying (Mobile/Desktop Apps)**

**Muitas vulns tão nos apps, não no web:**

```bash
# Setup:
1. Burp Suite proxy
2. Certificate install no device
3. Proxy mobile/desktop traffic

# Targets:
- APIs internas (não documentadas)
- Versões antigas de endpoints
- Debug features
- Hardcoded secrets
```

#### 9. **Rate Limiting = \$$\$$**

**Teste SEMPRE:**

* Brute force attacks
* Race conditions
* Resource exhaustion
* Mass assignment

**Tools:**

```bash
# Turbo Intruder (Burp)
# Custom Python scripts
# ffuf with high concurrency
```

#### 10. **Shodan/Censys (Infra em Scope)**

```bash
# Se infrastructure está no scope:
shodan search org:"Target Company"

# Procura por:
- Open databases (MongoDB, Elasticsearch)
- Exposed admin panels
- Default credentials
- Outdated services (CVEs)
- Cloud storage (S3, Azure Blob)
```

***

### 🎓 SKILLS QUE SEPARAM JUNIOR DE SENIOR

#### Technical Skills

| Skill              | Junior            | Senior                              |
| ------------------ | ----------------- | ----------------------------------- |
| **Report Writing** | "Found XSS"       | Business impact + POC + remediation |
| **Recon**          | subfinder + amass | Custom automation + monitoring      |
| **Exploitation**   | Single vuln       | Chaining multiple vulns             |
| **Tools**          | Usa tools         | Cria tools custom                   |
| **Scope**          | Web app only      | Web + Mobile + API + Infrastructure |

#### Business Skills (O DIFERENCIAL)

1. **Comunicação profissional** (email, LinkedIn, calls)
2. **Proposal writing** (consultoria, SOW)
3. **Networking** (conferências, Twitter, LinkedIn)
4. **Pricing strategy** (quanto cobrar?)
5. **Contract negotiation** (freelance/consulting)

***

### 📈 MÉTRICAS DE SUCESSO

#### KPIs Reais

```
JUNIOR:
├── 5-10 bugs/mês
├── $500-2k/mês
└── Public programs only

INTERMEDIATE:
├── 10-20 bugs/mês
├── $2k-5k/mês
├── Some private programs
└── Occasional consulting

SENIOR (TOP 1%):
├── 5-15 HIGH/CRITICAL bugs/mês
├── $10k-50k+/mês
├── Private programs + direct contracts
├── Recurring consulting clients
└── Platform reputation (top 100)
```

#### Track de Progresso

```markdown
## Monthly Tracker

### Stats
- Programs tested: X
- Bugs found: Y
- Total payout: $Z
- Consulting contracts: N
- New programs alerted: M

### Analysis
- Best vulnerability types this month
- Most profitable programs
- Time spent vs payout
- Skills to improve
```

***

### 🚀 AÇÃO IMEDIATA (SETUP EM 1 DIA)

#### Checklist Operacional

```markdown
- [ ] Configurar monitoramento de novos programas
  - [ ] BBScope instalado e configurado
  - [ ] Bounty Targets Data clone + cron
  - [ ] Telegram bot criado
  
- [ ] Configurar monitoramento de aquisições
  - [ ] RSS feeds adicionados (TechCrunch, Crunchbase)
  - [ ] Script de scraping funcionando
  - [ ] Notificações testadas
  
- [ ] Configurar monitoramento de subdomínios
  - [ ] Sublert instalado
  - [ ] crt.sh script pronto
  - [ ] Amass track configurado
  
- [ ] Preparar templates
  - [ ] Report template (Markdown)
  - [ ] Consulting proposal email
  - [ ] Follow-up email template
  
- [ ] Arsenal atualizado
  - [ ] Todas ferramentas instaladas e testadas
  - [ ] Burp extensions configuradas
  - [ ] Wordlists customizadas prontas
  
- [ ] Workflow documentado
  - [ ] Scripts de automação testados
  - [ ] Pipeline de recon definida
  - [ ] Checklist de testes pronta
```

***

### 🔗 RECURSOS ESSENCIAIS

#### Plataformas

* HackerOne: https://hackerone.com
* Bugcrowd: https://bugcrowd.com
* Intigriti: https://intigriti.com
* YesWeHack: https://yeswehack.com
* OpenBugBounty: https://openbugbounty.org

#### Ferramentas (GitHub)

```bash
# Recon
https://github.com/projectdiscovery/subfinder
https://github.com/OWASP/Amass
https://github.com/tomnomnom/httprobe
https://github.com/projectdiscovery/httpx
https://github.com/projectdiscovery/nuclei

# Monitoring
https://github.com/yassineaboukir/sublert
https://github.com/sw33tLie/bbscope
https://github.com/arkadiyt/bounty-targets-data

# Automation
https://github.com/pry0cc/axiom
https://github.com/codingo/Interlace

# Wordlists
https://github.com/danielmiessler/SecLists
https://github.com/assetnote/wordlists
```

#### Comunidades

* Twitter: #bugbounty #bugbountytips
* Discord: Bug Bounty Forum, Nahamsec
* Reddit: r/bugbounty
* YouTube: Nahamsec, STÖK, InsiderPhD

***

### 💎 GOLDEN RULES

1. **VELOCIDADE > PROFUNDIDADE** (primeiras 48h)
2. **NOVOS ALVOS > ALVOS SATURADOS**
3. **RELACIONAMENTO > VOLUME DE BUGS**
4. **CONSULTORIA > BUG BOUNTY PURO**
5. **AUTOMAÇÃO > TRABALHO MANUAL**
6. **BUSINESS SKILLS > TECHNICAL SKILLS** (long-term)
7. **QUALIDADE DO REPORT > QUANTIDADE**
8. **PRIVATE PROGRAMS > PUBLIC**
9. **NETWORK OFFLINE > ONLINE**
10. **CRIA FERRAMENTAS > SÓ USA FERRAMENTAS**

***

### 🎯 PRÓXIMOS 30 DIAS

#### Week 1: Setup

* \[ ] Implementar monitoring completo
* \[ ] Testar em 1 programa novo
* \[ ] Documentar processo

#### Week 2-3: Execution

* \[ ] 3-5 novos programas testados
* \[ ] Primeiros reports submetidos
* \[ ] Refinar workflow baseado em resultados

#### Week 4: Scale

* \[ ] Tentativa de upsell (se bugs confirmados)
* \[ ] Networking no LinkedIn
* \[ ] Análise de métricas e ajustes

***

**LEMBRE-SE:**

> "Os top 1% não são mais inteligentes. Eles são mais RÁPIDOS, mais FOCADOS e melhor CONECTADOS."

