# Bug Bounty

## Checklist Completo de Vulnerabilidades para Bug Bounty - Aplicações Web 2025

### Introdução

Este documento é um guia de referência abrangente para testers, bug bounty hunters e pentesters que desejam garantir cobertura máxima ao testar aplicações web. Baseado nas listas oficiais atualizadas (OWASP Top 10 2025, OWASP API Security Top 10, CWE Top 25), metodologias consagradas (PTES, NIST 800-115, OWASP WSTG) e práticas de hunters profissionais.

***

### 1. THREAT MODELING - O PRIMEIRO PASSO

Antes de começar o teste, realize um threat modeling para entender o escopo, arquitetura e riscos específicos da aplicação.

#### 1.1 Metodologias de Threat Modeling Recomendadas

**STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)**

* **Ferramenta oficial**: Microsoft Threat Modeling Tool
* **Uso**: Ideal para análise de fluxo de dados (DFD - Data Flow Diagrams)
* **Aplicação**: Web apps tradicionais, arquiteturas multi-camadas
* **Passo 1**: Diagram (criar arquitetura)
* **Passo 2**: Identify (listar ameaças com STRIDE)
* **Passo 3**: Mitigate (implementar controles)
* **Passo 4**: Validate (validar efetividade)

**PASTA (Process for Attack Simulation and Threat Analysis)**

* **Uso**: Mais orientado a risco de negócio
* **Aplicação**: Melhor para lógica de negócios complexa
* **Vantagem**: Foca em impacto real vs. apenas probabilidade técnica

**DREAD (Damage, Reproducibility, Exploitability, Affected users, Discoverability)**

* **Uso**: Priorização de ameaças identificadas
* **Complementa**: STRIDE ou PASTA

#### 1.2 Ferramentas de Threat Modeling

| Ferramenta                     | Tipo       | Preço         | Melhor Para             |
| ------------------------------ | ---------- | ------------- | ----------------------- |
| Microsoft Threat Modeling Tool | Desktop    | Gratuito      | STRIDE methodology      |
| IriusRisk                      | SaaS       | Pago          | DevSecOps, automation   |
| OWASP Threat Dragon            | Web        | Gratuito      | Diagramming rápido      |
| Threagile                      | CLI        | Gratuito      | Infrastructure-as-code  |
| ThreatModeler                  | Enterprise | Pago          | Grandes organizações    |
| SecuriCAD                      | Academic   | Gratuito/Pago | Análise visual avançada |

#### 1.3 Checklist Pré-Teste (Threat Modeling)

* [ ] Mapear todos os endpoints da aplicação (web, API, WebSocket)
* [ ] Identificar fluxos de dados (entrada → processamento → saída)
* [ ] Listar limites de confiança (trust boundaries)
* [ ] Identificar pontos de autenticação/autorização
* [ ] Mapear integração com serviços terceiros
* [ ] Documentar tecnologias stack (linguagem, frameworks, banco de dados)
* [ ] Listar funcionalidades críticas (transações, dados sensíveis)
* [ ] Identificar pontos de acesso administrativo
* [ ] Mapear fluxos de dados pessoais/sensíveis
* [ ] Aplicar STRIDE em cada componente
* [ ] Priorizar ameaças com DREAD
* [ ] Documentar cenários de ataque potenciais

***

### 2. OWASP TOP 10 2025 - VULNERABILIDADES WEB APPLICATION

Baseado em análise de CVE data 2021-2024 e tendências atuais.

#### **A01:2025 - Broken Access Control** (Controle de Acesso Quebrado)

**Descrição**: Usuários podem acessar recursos, dados ou funcionalidades além de suas permissões.

**Subcategorias para testar**:

**2.1.1 IDOR (Insecure Direct Object Reference)**

```
Teste: Alterar IDs em URLs/requests para acessar recursos de outros usuários
GET /api/users/123/profile
→ Tente: GET /api/users/124/profile, /api/users/999/profile

Teste: Alterar IDs ocultos em JSON/parâmetros POST
POST /api/invoices/view
{"invoice_id": 1001}
→ Tente: {"invoice_id": 1002}, {"invoice_id": 9999}

Checklist IDOR:
- [ ] Testar modificação de ID numéricos sequenciais
- [ ] Testar UUID/GUIDs em múltiplos formatos
- [ ] Testar parâmetros hash (MD5, SHA1)
- [ ] Testar parâmetros codificados (Base64, URL-encoded)
- [ ] Testar IDs em URLs, query params, POST bodies, headers, cookies
- [ ] Testar acesso horizontal (mesmo nível de privilégio)
- [ ] Testar acesso vertical (privilégio menor → maior)
- [ ] Verificar se aplicação retorna dados sensíveis vs. apenas erro 403
- [ ] Testar com múltiplas contas (admin, user, guest)
- [ ] Testar race conditions em acesso paralelo (race-based IDOR)
```

**2.1.2 Privilege Escalation (Horizontal & Vertical)**

```
Horizontal: User A acessa dados/funcionalidades de User B (mesmo nível)
Vertical: User normal executa ações de Admin

Teste: Modificar parâmetros de privilégio
POST /admin/create-user
{"username": "attacker", "role": "admin"}

Teste: JWT com role modificado
Original: {"user_id": 123, "role": "user"}
Modificado: {"user_id": 123, "role": "admin"}

Checklist Privilege Escalation:
- [ ] Editar claims em JWT/tokens
- [ ] Alterar parâmetros de role em requests
- [ ] Alterar hidden fields em formulários
- [ ] Modificar cookies de sessão
- [ ] Testar acesso direto a endpoints admin (/admin, /management)
- [ ] Verificar validação de role server-side vs. client-side
- [ ] Testar bypass via método HTTP (GET vs. POST)
- [ ] Testar parameter pollution (role=user&role=admin)
- [ ] Verificar se há checks de autorização em CADA ação sensível
- [ ] Testar autorização baseada em referrer/origin
- [ ] Testar CORS bypass para privilege escalation
```

**2.1.3 Path Traversal / Directory Traversal**

```
../../../etc/passwd
..%2F..%2F..%2Fetc%2Fpasswd
....//....//....//etc/passwd
..\ (Windows)
Duplos encoding: %252e%252e%252f

Checklist:
- [ ] Testar ../../../ variações
- [ ] Testar null bytes: ..%00/
- [ ] Testar Unicode encoding
- [ ] Testar double URL encoding
- [ ] Testar case variation (../ vs ..\\)
- [ ] Testar com trailing caracteres: ../../../etc/passwd%00.jpg
- [ ] Testar em file upload (path traversal em nome)
- [ ] Testar em parâmetros de arquivo (file=, download=, path=)
- [ ] Verificar com diferentes extensões (.pdf, .txt, .php)
- [ ] Testar chroot/jail bypass
- [ ] Testar symlink following
- [ ] Testar em APIs (JSON parameters)
```

**2.1.4 Funcionalidades Administrativas Expostas**

```
Checklist:
- [ ] Procurar endpoints /admin, /manager, /superadmin, /staff
- [ ] Verificar arquivo robots.txt para caminhos sensíveis
- [ ] Procurar em JavaScript/source code URLs administrativas
- [ ] Testar acesso direto sem autenticação
- [ ] Testar com autenticação de usuário normal
- [ ] Procurar debug panels (Django debug=True, Spring Boot Actuator)
- [ ] Verificar backup files (.bak, .old, .backup)
- [ ] Procurar test/staging endpoints expostos
- [ ] Verificar git/.git exponho
- [ ] Procurar config files exponho (.env, config.php)
```

**2.1.5 Método HTTP Override**

```
POST /delete?_method=DELETE
X-HTTP-Method-Override: PUT
X-Method-Override: DELETE

Checklist:
- [ ] Testar X-HTTP-Method-Override header
- [ ] Testar X-Method-Override header
- [ ] Testar _method parameter
- [ ] Testar method parameter
- [ ] Verificar se PUT/DELETE são bloqueados mas GET/POST não
- [ ] Testar CSRF via method override
```

**2.1.6 CORS Misconfiguration**

```
Origin: https://attacker.com
→ Response: Access-Control-Allow-Origin: https://attacker.com

Checklist:
- [ ] Testar Origin: null
- [ ] Testar Origin: arbitrário (attacker.com)
- [ ] Testar Origin: target.attacker.com
- [ ] Testar Origin: target.com.attacker.com
- [ ] Verificar wildcard (*) em Access-Control-Allow-Origin
- [ ] Verificar credentials em requisições CORS (credenciais: true)
- [ ] Testar regex fraco em validação (.*target.com)
- [ ] Verificar Access-Control-Allow-Methods (PUT, DELETE?)
- [ ] Verificar Access-Control-Expose-Headers
- [ ] Testar preflight request bypass
```

***

#### **A02:2025 - Injection** (Injeção)

**Descrição**: Dados não confiáveis são passados para interpretadores (SQL, OS, template, etc).

**2.2.1 SQL Injection (SQLi)**

**Tipos**:

* Erro-baseado (Union-based)
* Boolean-baseado (Blind)
* Time-baseado (Blind)
* Stacked queries
* Second-order SQL injection

```
Payloads básicos:
' OR '1'='1
' OR 1=1--
' UNION SELECT NULL,NULL,NULL--
'; DROP TABLE users;--
' AND SLEEP(5)--

Checklist SQL Injection:
- [ ] Testar em parâmetros GET/POST
- [ ] Testar em headers (User-Agent, Referer, X-Forwarded-For)
- [ ] Testar em cookies
- [ ] Testar em dados JSON
- [ ] Testar em XML
- [ ] Testar em GraphQL queries
- [ ] Verificar em ORDER BY (ORDER BY 1,2,3...)
- [ ] Testar UNION-based (detectar número de colunas)
- [ ] Testar Boolean-based (if/else logic)
- [ ] Testar time-based (SLEEP, BENCHMARK)
- [ ] Testar out-of-band (DNS exfiltration, HTTP callback)
- [ ] Verificar WAF/filtros e técnicas de bypass
- [ ] Testar encoding: double URL, Unicode, Hex
- [ ] Testar casos: MySQL, PostgreSQL, MSSQL, Oracle, SQLite
- [ ] Testar em stored procedures/views
- [ ] Verificar blind SQLi via HTTP response time
- [ ] Testar stacked queries (';SELEC T...;)
- [ ] Testar second-order injection (dados salvos → usados depois)
```

**Ferramentas**: sqlmap, Burp Suite Pro, manual testing com Burp Repeater

**2.2.2 NoSQL Injection**

```
MongoDB:
{"username": {"$ne": null}, "password": {"$ne": null}}
{"username": {"$regex": "^admin"}}

Checklist:
- [ ] Testar em queries MongoDB ($where, $ne, $regex)
- [ ] Testar em CouchDB
- [ ] Testar em Redis
- [ ] Testar authentication bypass com NoSQL
- [ ] Testar boolean-based blind NoSQL
- [ ] Testar time-based blind NoSQL
- [ ] Verificar em APIs REST que usam NoSQL
```

**2.2.3 Command Injection (OS Command Injection)**

```
; id
| whoami
&& cat /etc/passwd
` cmd.exe /c whoami `
$( ping -c 1 127.0.0.1 )

Checklist:
- [ ] Testar em parâmetros de formulário
- [ ] Testar em upload de arquivo
- [ ] Testar em parâmetros de API
- [ ] Verificar caracteres especiais: ;, |, &, `, $(), <>, >
- [ ] Testar blind command injection (time-based)
- [ ] Testar out-of-band command injection (DNS, HTTP callback)
- [ ] Verificar contexto: bash, cmd.exe, powershell
- [ ] Testar filter bypass (blacklist evasion)
- [ ] Verificar PATH traversal em binários
```

**Ferramentas**: Burp Intruder, manual testing

**2.2.4 LDAP Injection**

```
*)(|(uid=*
admin*

Checklist:
- [ ] Testar em login/autenticação LDAP
- [ ] Verificar caracteres especiais: *, (, ), &, |
- [ ] Testar filter injection
- [ ] Verificar wildcard matching
```

**2.2.5 XPath Injection**

```
' OR '1'='1
admin' or 1=1 or '

Checklist:
- [ ] Testar em consultas XML
- [ ] Verificar em autenticação baseada em XPath
- [ ] Testar boolean-based blind XPath injection
```

**2.2.6 Template Injection (SSTI - Server-Side Template Injection)**

```
${7*7}
#{7*7}
<%= 7*7 %>
{{7*7}}
{[7*7]}

Engines:
- Jinja2 (Python): {{7*7}}, {{config}}
- ERB (Ruby): <%= 7*7 %>
- Handlebars: {{7*7}}
- Freemarker: <#assign value=7*7>
- Velocity: #set($x=7*7)

Checklist:
- [ ] Identificar template engine (Jinja2, Freemarker, Velocity, Handlebars, etc)
- [ ] Testar syntax básica de math: {{7*7}}
- [ ] Escalar para acesso a variáveis globais
- [ ] Escalar para RCE (remote code execution)
- [ ] Testar em emails (velocity, freemarker)
- [ ] Testar em PDFs (freemarker, velocity)
- [ ] Verificar sandbox escape
- [ ] Testar em múltiplos pontos (subject, body, headers)
```

**Ferramentas**: tplmap, manual Burp testing

**2.2.7 XXE (XML External Entity)**

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>&xxe;</root>

Tipos XXE:
- Classic XXE (file disclosure)
- Blind XXE (via error-based ou time-based)
- XXE via parameter entity
- XXE + SSRF
- XXE + XXE Denial of Service (Billion Laughs)

Checklist:
- [ ] Testar em XML upload
- [ ] Testar em SOAP requests
- [ ] Testar em SVG upload (XXE via SVG)
- [ ] Testar em Office documents (DOCX, XLSX, PPTX - são ZIPs com XMLs)
- [ ] Testar em PDF upload
- [ ] Verificar if DOCTYPE é permitido
- [ ] Testar SYSTEM entity
- [ ] Testar PUBLIC entity
- [ ] Testar parameter entities para blind XXE
- [ ] Testar XXE + SSRF (internal network scanning)
- [ ] Testar XXE denial of service
- [ ] Verificar proteção: libxml2, disable_entity_loader()
```

**Ferramentas**: XXE injector, Burp Pro, manual testing

**2.2.8 XPath Injection**

```
' OR '1'='1
admin' or '1'='1
*

Checklist:
- [ ] Testar em login XPath-based
- [ ] Testar boolean blind XPath
- [ ] Testar time-based blind XPath
- [ ] Verificar response differences
```

**2.2.9 CSTI (Client-Side Template Injection)**

```
AngularJS: {{7*7}}, ${{7*7}}
Vue: {{7*7}}
React: (geralmente safe se usar JSX)

Checklist:
- [ ] Testar em aplicações AngularJS/Vue/etc
- [ ] Identificar template engine client-side
- [ ] Escalar para XSS via CSTI
- [ ] Testar sandbox escape
```

**2.2.10 Expression Language (EL) Injection**

```
${7*7}
#{7*7}
%{7*7}

Checklist:
- [ ] Testar em JSP pages
- [ ] Testar em Spring Expression Language
- [ ] Testar acesso a classes Java
```

***

#### **A03:2025 - Insecure Design** (Projeto Inseguro)

**Descrição**: Vulnerabilidades de lógica de negócio, design flaws, falta de controles de segurança.

**2.3.1 Business Logic Flaws**

```
Exemplos:
- Comprar produto com preço negativo
- Reutilizar coupon ilimitadamente
- Pular passos em multi-step workflow
- Aplicar múltiplos descontos
- Manipular quantidade em carrinho
- Race conditions em transações

Checklist:
- [ ] Mapear todo workflow da aplicação
- [ ] Testar bypass de validação client-side
- [ ] Testar acesso direto a passos intermediários
- [ ] Testar reordenação de passos
- [ ] Testar parâmetros numéricos com valores negativos
- [ ] Testar parâmetros com valores extremamente grandes
- [ ] Testar valores float vs. integer
- [ ] Testar reutilização de tokens/vouchers
- [ ] Testar aplicação múltipla de mesma ação
- [ ] Testar race conditions (requisições paralelas)
- [ ] Testar manipulação de quantidade/inventory
- [ ] Testar double-spending em créditos
- [ ] Testar price manipulation em checkout
- [ ] Verificar se há rate limiting em ações críticas
- [ ] Testar manipulação de datas (expiration, activation)
- [ ] Testar acesso a recursos expirados
- [ ] Testar contaminação de estado (cross-user)
```

**2.3.2 Missing Security Controls**

```
Checklist:
- [ ] Rate limiting ausente em login (brute force)
- [ ] Rate limiting ausente em API endpoints
- [ ] Falta de validação de email (no confirmation)
- [ ] Falta de 2FA em ações sensíveis
- [ ] Falta de audit logging
- [ ] Falta de anomaly detection
- [ ] Falta de CSRF tokens
- [ ] Falta de input validation
- [ ] Falta de encryption em dados sensíveis
- [ ] Falta de certificate pinning (mobile)
- [ ] Falta de security headers
```

**2.3.3 Workflow Bypass**

```
Exemplo: Compra de produto
Passo 1: Seleção → Passo 2: Pagamento → Passo 3: Confirmação

Ataque: Acessar Passo 3 diretamente sem Passo 2

Checklist:
- [ ] Testar acesso direto a passos intermediários
- [ ] Testar pulo de passos
- [ ] Testar reordenação de passos
- [ ] Verificar se estado é validado server-side
- [ ] Testar com sessões paralelas
- [ ] Testar com múltiplas abas do browser
```

***

#### **A04:2025 - Identification and Authentication Failures** (Falhas em Autenticação)

**2.4.1 Weak Password Policy**

```
Checklist:
- [ ] Testar login com senhas fracas (admin/admin, password)
- [ ] Verificar requisitos de senha (comprimento mínimo)
- [ ] Verificar complexidade (maiúsc, minúsc, números, símbolos)
- [ ] Testar reutilização de senhas antigas
- [ ] Verificar histórico de senhas
- [ ] Testar reset de senha vulnerável
- [ ] Testar password hints exponho
```

**2.4.2 Credential Brute Force / Account Enumeration**

```
Checklist:
- [ ] Testar rate limiting em login
- [ ] Testar account lockout após N tentativas falhadas
- [ ] Verificar enumeração de usuários (erro diferente para user vs. password)
- [ ] Testar brute force em esquecimento de senha
- [ ] Testar brute force em 2FA
- [ ] Testar credential stuffing (reuse from breached databases)
- [ ] Verificar proteção contra bot (CAPTCHA)
```

**Ferramentas**: Burp Intruder, hashcat, John the Ripper, Hydra

**2.4.3 Session Management Flaws**

```
Checklist:
- [ ] Verificar previsibilidade de Session ID
- [ ] Testar fixação de sessão (forçar session_id específico)
- [ ] Verificar timeout de sessão
- [ ] Testar invalidação ao logout
- [ ] Verificar CSRF tokens em state-changing requests
- [ ] Testar session hijacking
- [ ] Verificar validação de User-Agent/IP
- [ ] Testar concurrent sessions (múltiplos logins simultâneos)
- [ ] Verificar cookie flags (Secure, HttpOnly, SameSite)
- [ ] Testar session regeneration após login
```

**2.4.4 Weak Cryptography / Reversible Encryption**

```
Checklist:
- [ ] Verificar se senhas são salted
- [ ] Testar hash strength (MD5, SHA1 = fraco)
- [ ] Verificar se PBKDF2, bcrypt, scrypt são usados
- [ ] Testar se dados são criptografados com algoritmos fracos
- [ ] Verificar chaves de criptografia hardcoded
- [ ] Testar chaves default conhecidas
- [ ] Verificar ECB mode (determinístico = ruim)
- [ ] Testar chaves geradas de senhas fracas
```

**2.4.5 Default Credentials**

```
Checklist:
- [ ] Testar admin/admin, admin/password, root/root
- [ ] Testar credenciais padrão conhecidas (framework-specific)
- [ ] Verificar contas de teste não removidas
- [ ] Testar backup accounts
- [ ] Procurar credentials em documentação/código
- [ ] Verificar console de administração com defaults
```

**2.4.6 Authentication Bypass**

```
Checklist:
- [ ] Testar acesso direto a URLs autenticadas (sem login)
- [ ] Testar modificação de URL (remove /login)
- [ ] Testar parameter pollution (auth=yes&auth=no)
- [ ] Testar null byte injection (/admin%00.jpg)
- [ ] Testar case variation (/ADMIN)
- [ ] Testar método HTTP alternative (GET vs. POST)
- [ ] Testar if-condition bypass
- [ ] Testar cookie deletion/bypass
- [ ] Testar session fixation
- [ ] Testar referrer-based auth
- [ ] Testar básico auth encoding bypass
```

**2.4.7 Insecure Password Reset**

```
Checklist:
- [ ] Testar previsibilidade de reset token
- [ ] Verificar expiração de reset token
- [ ] Testar reutilização de token
- [ ] Verificar se token é invalidado após uso
- [ ] Testar IDOR em reset URL (/reset?token=ABC&user_id=123)
- [ ] Testar se email é validado antes de reset
- [ ] Verificar information disclosure (confirma/nega existência de conta)
- [ ] Testar reset link em múltiplos navegadores/devices
- [ ] Verificar se aceita senhas fracas em reset
- [ ] Testar race condition (múltiplos resets simultâneos)
```

**2.4.8 Multi-Factor Authentication Bypass**

```
Checklist:
- [ ] Testar bypass de 2FA enviando requisição sem 2FA
- [ ] Testar previsibilidade de TOTP
- [ ] Testar brute force em código 2FA
- [ ] Verificar expiração de código 2FA
- [ ] Testar backup codes reutilização
- [ ] Verificar se 2FA é obrigatório mesmo após login
- [ ] Testar race condition entre login e 2FA check
- [ ] Verificar se 2FA pode ser desativado sem verificação
- [ ] Testar logout + login direto (pulando 2FA)
- [ ] Verificar se SMS 2FA sofre SIM swapping
```

**2.4.9 OAuth Weaknesses**

```
Checklist:
- [ ] Testar OAuth redirect URI validation
- [ ] Verificar se qualquer redirect_uri é aceito
- [ ] Testar open redirect via OAuth callback
- [ ] Verificar se authorization code é reutilizável
- [ ] Verificar expiração de authorization code
- [ ] Testar if state parameter é validado
- [ ] Verificar PKCE em OAuth flow
- [ ] Testar token leakage em logs/referrer
- [ ] Verificar se token é enviado apenas via POST (não GET)
- [ ] Verificar scopes solicitados (overly broad?)
- [ ] Testar CSRF em desligar OAuth
- [ ] Verificar validação de issuer token
```

**2.4.10 JWT Vulnerabilities**

```
Checklist:
- [ ] Testar alteração de algoritmo (RS256 → HS256)
- [ ] Verificar se server valida assinatura
- [ ] Testar remoção de assinatura ("none" algorithm)
- [ ] Verificar se secret é fraco (brute-forceable)
- [ ] Testar modificação de claims (role, user_id)
- [ ] Verificar expiração token (exp claim)
- [ ] Testar reutilização de tokens revogados
- [ ] Verificar se jku (JWK URL) é validado
- [ ] Testar key injection attacks
- [ ] Verificar se token é armazenado seguramente (httpOnly?)
- [ ] Testar XSS para roubo de JWT
- [ ] Verificar se servidor rejeita tokens muito antigos
- [ ] Testar reuse de nonce em algoritmos ECDSA
```

**Ferramentas**: jwt.io, jwt\_tool.py, hashcat

***

#### **A05:2025 - Cryptographic Failures** (Falhas Criptográficas)

**2.5.1 Data Exposure in Transit**

```
Checklist:
- [ ] Verificar HTTPS em todas as páginas
- [ ] Testar mixed content (HTTP + HTTPS)
- [ ] Verificar cipher suites fortes (TLS 1.2+)
- [ ] Verificar certificado válido (domínio, CA, validade)
- [ ] Testar downgrade attack (força HTTP)
- [ ] Verificar HSTS header
- [ ] Testar certificate pinning (mobile)
- [ ] Verificar se dados sensíveis são transmitidos em plaintext
- [ ] Verificar protocol weak (SSLv2, SSLv3, TLS 1.0, TLS 1.1)
```

**Ferramentas**: SSLLabs, testssl.sh, Burp

**2.5.2 Data Exposure at Rest**

```
Checklist:
- [ ] Verificar se senhas são hashed (não plaintext)
- [ ] Verificar se dados PII são criptografados
- [ ] Testar se chave de criptografia é hardcoded
- [ ] Verificar força de hash (bcrypt, scrypt, Argon2)
- [ ] Verificar se backup é criptografado
- [ ] Testar se keys estão em version control (.git)
- [ ] Verificar permissions de arquivo (chmod)
- [ ] Testar acesso a banco de dados backup
- [ ] Verificar se logs contêm dados sensíveis
```

**2.5.3 Weak Cryptographic Algorithms**

```
Checklist:
- [ ] Testar uso de MD5, SHA1 (fraco para senhas)
- [ ] Verificar se DES, RC4 são usados (deprecated)
- [ ] Verificar ECB mode (determinístico)
- [ ] Testar chaves curtas (< 128 bits)
- [ ] Verificar generation de chaves aleatórias (não determinística)
- [ ] Testar se IV é reutilizado
- [ ] Verificar chaves derivadas de senhas (usar PBKDF2, não simples hash)
```

**2.5.4 Information Disclosure via Errors**

```
Checklist:
- [ ] Verificar stack traces exponho
- [ ] Testar banco de dados error messages exponho
- [ ] Verificar file paths exponho
- [ ] Testar versão de software exponho
- [ ] Verificar comentários de código exponho (source maps)
- [ ] Testar default error pages vs. customizado
- [ ] Verificar se logs exponho dados sensíveis
```

***

#### **A06:2025 - Security Misconfiguration** (Configuração Insegura)

**2.6.1 Default Credentials / Accounts**

```
Checklist:
- [ ] Testar admin/admin, root/root padrões
- [ ] Verificar contas de teste não removidas (demo, test)
- [ ] Testar credenciais padrão de frameworks conhecidos
- [ ] Procurar em documentação padrão (dentro do código)
```

**2.6.2 Unnecessary Features Enabled**

```
Checklist:
- [ ] Verificar if debug mode é ativado em produção
- [ ] Testar Spring Boot Actuator endpoints
- [ ] Verificar Django DEBUG=True
- [ ] Testar endpoints de desenvolvimento exponho
- [ ] Verificar if admin panels estão públicos
- [ ] Testar if swagger/OpenAPI está público
- [ ] Verificar if SOAP endpoints estão públicos
```

**2.6.3 Insecure HTTP Headers**

```
Checklist:
- [ ] Verificar if Content-Security-Policy (CSP) está presente
- [ ] Testar X-Frame-Options (clickjacking protection)
- [ ] Verificar X-Content-Type-Options: nosniff
- [ ] Testar Strict-Transport-Security (HSTS)
- [ ] Verificar X-XSS-Protection
- [ ] Testar Referrer-Policy
- [ ] Verificar Cache-Control em dados sensíveis
- [ ] Testar Permissions-Policy (Feature-Policy)
```

**2.6.4 CORS Misconfiguration**

```
(Já coberto em Broken Access Control, mas incluir aqui também)

Checklist:
- [ ] Verificar Access-Control-Allow-Origin: *
- [ ] Testar CORS com credentials
- [ ] Verificar wildcard em credenciais
- [ ] Testar regex fraco (.*example.com)
```

**2.6.5 Information Disclosure**

```
Checklist:
- [ ] Testar robots.txt / sitemap.xml
- [ ] Verificar .git / .svn exponho
- [ ] Testar .env files
- [ ] Verificar backup files (.bak, .old)
- [ ] Testar source maps (.js.map)
- [ ] Procurar em bucket S3 aberto (AWS)
- [ ] Verificar Google Cloud Storage bucket
- [ ] Testar Azure Blob Storage
- [ ] Verificar package manager files (composer.json, package.json, pom.xml)
```

**2.6.6 Server Misconfiguration**

```
Checklist:
- [ ] Testar directory listing (Index of /)
- [ ] Verificar versão do servidor exponho
- [ ] Testar métodos HTTP desnecessários (PUT, DELETE)
- [ ] Verificar if WebDAV está ativado
- [ ] Testar HTTP TRACE method
- [ ] Verificar se CORS pode ser abusado
```

**2.6.7 Database Misconfiguration**

```
Checklist:
- [ ] Verificar if banco está acessível externamente
- [ ] Testar credenciais padrão do banco
- [ ] Verificar if query logging está desativado (pode expor dados)
- [ ] Testar replicação não criptografada
- [ ] Verificar backup não criptografado
```

***

#### **A07:2025 - Vulnerable and Outdated Components** (Componentes Vulneráveis)

**2.7.1 Known Vulnerabilities in Dependencies**

```
Checklist:
- [ ] Executar npm audit (JavaScript)
- [ ] Executar pip check (Python)
- [ ] Executar bundle audit (Ruby)
- [ ] Usar OWASP Dependency-Check
- [ ] Usar Snyk para scanning
- [ ] Procurar CVE em package manager conhecidos
- [ ] Verificar versões de frameworks (Rails, Django, Spring, etc)
- [ ] Testar exploits de vulnerabilidades conhecidas
- [ ] Verificar transitive dependencies
- [ ] Verificar se patches foram aplicados
```

**Ferramentas**: npm audit, OWASP Dependency-Check, Snyk, Black Duck

**2.7.2 Outdated Software**

```
Checklist:
- [ ] Procurar versão do servidor (Apache, Nginx, IIS)
- [ ] Identificar versão de framework
- [ ] Testar se patches de segurança foram aplicados
- [ ] Verificar end-of-life software ainda em uso
- [ ] Procurar vulnerabilidades específicas de versão
```

**2.7.3 Supply Chain Attacks**

```
Checklist:
- [ ] Verificar integridade de packages (hash verificação)
- [ ] Verificar assinatura de packages
- [ ] Testar se pode fazer package injection
- [ ] Verificar se build process é seguro
- [ ] Verificar CI/CD pipeline security
- [ ] Testar compromiso de repositório
```

***

#### **A08:2025 - Software and Data Integrity Failures** (Falhas de Integridade)

**2.8.1 Insecure Deserialization**

```
Java serialization: AC ED 00 05 (hex), rO0 (base64)
Python pickle: 80 04 95 (protocolo 4)

Checklist:
- [ ] Procurar serialização de objetos (Java, Python, PHP)
- [ ] Testar gadget chains (ysoserial)
- [ ] Verificar se entrada de usuário é desserializada
- [ ] Procurar em cookies, sessão, parâmetros
- [ ] Testar RCE via deserialization
- [ ] Verificar proteção (whitelist de classes)
- [ ] Procurar em XML, JSON parsing (se faz object instantiation)
```

**Ferramentas**: ysoserial, marshalsec

**2.8.2 Insecure CI/CD**

```
Checklist:
- [ ] Verificar se código sem review é mergeado
- [ ] Testar if pode fazer code injection em build
- [ ] Verificar if secrets estão em git history
- [ ] Testar if pode fazer supply chain attack
- [ ] Verificar assinatura de artifacts
- [ ] Procurar webhooks não validados
```

**2.8.3 Unsigned Updates**

```
Checklist:
- [ ] Verificar se updates são assinados
- [ ] Testar if pode fazer MITM em update
- [ ] Procurar for unsigned binary
- [ ] Verificar certificado de assinatura
```

***

#### **A09:2025 - Security Logging and Monitoring Failures** (Falhas em Logging)

**2.9.1 Missing or Inadequate Logging**

```
Checklist:
- [ ] Verificar se login é logado
- [ ] Testar se failed login attempts são logados
- [ ] Verificar acesso a dados sensíveis é logado
- [ ] Testar se admin actions são logados
- [ ] Procurar if error conditions são logadas
- [ ] Verificar retenção de logs (quanto tempo?)
- [ ] Testar se logs contêm stack traces (information disclosure)
```

**2.9.2 Insufficient Monitoring**

```
Checklist:
- [ ] Verificar alertas em multiple failed login
- [ ] Testar alertas em unusual activity
- [ ] Procurar for rate limiting bypass
- [ ] Verificar if brute force é detectado
- [ ] Testar if credential stuffing é detectado
```

***

#### **A10:2025 - Mishandling of Exceptional Conditions** (Antes: SSRF)

**Descrição**: Inclui SSRF, tratamento inadequado de exceções, input não validado em contextos especiais.

**2.10.1 Server-Side Request Forgery (SSRF)**

```
Targets:
http://localhost:8000
http://127.0.0.1
http://169.254.169.254/latest/meta-data/ (AWS metadata)
http://metadata.google.internal/computeMetadata/v1/
file:///etc/passwd

Bypass techniques:
http://127.0.0.1:8000
http://0.0.0.0
http://localhost.
http://127.1
http://2130706433 (numeric IP)
http://0177.0.0.1 (octal)
http://0x7f.0x0.0x0.0x1 (hex)

Checklist:
- [ ] Testar em parâmetros de URL (url=, link=, redirect=)
- [ ] Testar em upload de imagem (procura por URL)
- [ ] Testar em HTML parsing (fetch da URL)
- [ ] Testar em XML/SVG parsing
- [ ] Testar em PDF generation (wkhtmltopdf)
- [ ] Testar em webhook (callback URL)
- [ ] Verificar blacklist de IPs (localhost, 127.0.0.1, 169.254.x.x)
- [ ] Testar bypass de blacklist (numeric IP, octal, hex, DNS rebinding)
- [ ] Testar acesso a AWS metadata service
- [ ] Testar acesso a Google Cloud metadata
- [ ] Testar acesso a Azure metadata
- [ ] Testar SSRF + XXE (via out-of-band)
- [ ] Procurar em parâmetros que referenciam URLs
- [ ] Testar em redirect chain
- [ ] Testar time-based SSRF (demora na resposta)
```

**Ferramentas**: Burp Collaborator, Out-of-band techniques

**2.10.2 Open Redirect**

```
Checklist:
- [ ] Procurar parâmetros redirect_uri, return_to, continue
- [ ] Procurar em logout (redirect para URL)
- [ ] Testar em OAuth callback
- [ ] Verificar validação de domínio (whitelist)
- [ ] Testar bypass (//attacker.com, ///attacker.com)
- [ ] Testar data URLs (data:text/html,...)
- [ ] Testar javascript: URLs
- [ ] Procurar em response headers (Location)
- [ ] Testar em meta refresh
- [ ] Verificar if é usado em phishing
```

**2.10.3 Improper Error Handling**

```
Checklist:
- [ ] Verificar if stack traces são exponho
- [ ] Testar se error messages revelam informações
- [ ] Procurar por default error pages
- [ ] Verificar se database errors são exponho
- [ ] Testar if server version é exponho
- [ ] Procurar por desenvolvedor comments em errors
```

***

### 3. OWASP API SECURITY TOP 10 - 2023 (Aplicável para APIs Web)

**Nota**: Se a aplicação expõe APIs, estas vulnerabilidades devem ser testadas.

#### API1:2023 - Broken Object Level Authorization (BOLA)

```
Teste IDOR em APIs:
GET /api/v1/users/123/profile
→ Tente: /api/v1/users/124/profile

POST /api/v1/invoices
Body: {"invoice_id": 1001}
→ Tente: {"invoice_id": 1002}

Checklist:
- [ ] Testar modificação de ID em URL path
- [ ] Testar modificação de ID em query parameter
- [ ] Testar modificação de ID em request body (JSON)
- [ ] Testar com múltiplas contas
- [ ] Verificar validação server-side
```

#### API2:2023 - Broken Authentication

```
Checklist (igual a A04, específico para APIs):
- [ ] Verificar JWT validation
- [ ] Testar API key reuse
- [ ] Testar token expiration
- [ ] Verificar CORS + authentication
- [ ] Testar autenticação opcional
```

#### API3:2023 - Broken Object Property Level Authorization

```
Exemplo:
GET /api/users/123
Response:
{
  "user_id": 123,
  "username": "john",
  "email": "john@example.com",
  "salary": 50000,  // Não deve retornar
  "is_admin": false
}

Checklist:
- [ ] Procurar por sensitive fields na resposta
- [ ] Testar mass assignment (PUT/PATCH com campos extras)
- [ ] Verificar field-level authorization
- [ ] Procurar exposição de PII desnecessária
```

#### API4:2023 - Unrestricted Resource Consumption

```
Checklist:
- [ ] Testar rate limiting em endpoints
- [ ] Testar upload de arquivo muito grande
- [ ] Testar requisições de longa duração (timeout)
- [ ] Testar CPU-intensive operations
- [ ] Procurar por pagination bypass (limit=-1, limit=999999)
- [ ] Testar batch operations sem limite
```

#### API5:2023 - Broken Function Level Authorization

```
Checklist:
- [ ] Testar acesso a endpoints admin com user normal
- [ ] Procurar por /admin, /management, /api/admin
- [ ] Testar método HTTP alternative
- [ ] Verificar autorização em cada função
- [ ] Testar parameter pollution para escalação
```

#### API6:2023 - Unrestricted Access to Sensitive Business Flows

```
Exemplo: Limite de compras por hora
Ataque: Usar bot para comprar ilimitadamente

Checklist:
- [ ] Procurar por business logic não protegida
- [ ] Testar rate limiting em ações críticas
- [ ] Verificar limite de transações
- [ ] Testar captcha/challenge em ações repetidas
```

#### API7:2023 - Server Side Request Forgery

```
(Já coberto em A10, mas importante para APIs)
```

#### API8:2023 - Security Misconfiguration

```
Checklist:
- [ ] Testar Swagger/OpenAPI público (expõe endpoints)
- [ ] Verificar versão da API exponho
- [ ] Testar método HTTP não esperado (HEAD, TRACE)
- [ ] Procurar por default endpoints
- [ ] Verificar CORS

 aberto demais
```

#### API9:2023 - Improper Inventory Management

```
Checklist:
- [ ] Procurar por versões antigas da API (v1, v2)
- [ ] Testar endpoints deprecados ainda ativos
- [ ] Verificar acesso a beta APIs
- [ ] Procurar por endpoints não documentados
- [ ] Testar shadow APIs
```

#### API10:2023 - Unsafe Consumption of APIs

```
Checklist:
- [ ] Verificar validação de resposta de API terceira
- [ ] Testar injection via resposta de terceira
- [ ] Procurar por XML/JSON parsing inseguro
- [ ] Verificar deserialization de dados terceira
```

***

### 4. OWASP TESTING GUIDE (WSTG) - CATEGORIAS ADICIONAIS

Para cobertura ainda mais completa, incluir testes do OWASP WSTG:

#### 4.1 Information Gathering

```
Checklist:
- [ ] Google dorking (site:, inurl:, filetype:)
- [ ] Procurar em Wayback Machine (web.archive.org)
- [ ] DNS enumeration (dig, nslookup)
- [ ] WHOIS lookup
- [ ] Certificate transparency logs (crt.sh)
- [ ] GitHub reconnaissance (procurar URLs, commits, credenciais)
- [ ] Shodan / Censys (encontrar infraestrutura exposta)
- [ ] Website fingerprinting (teknologi stack)
- [ ] Buscar dados em search engines
```

#### 4.2 Configuration and Deployment

```
Checklist:
- [ ] Testar métodos HTTP desnecessários
- [ ] Verificar diretórios abertos
- [ ] Procurar files de backup exponho
- [ ] Verificar file extensions sensíveis
- [ ] Procurar por interfaces admin não documentadas
- [ ] Testar cloud storage (S3, GCS, Azure Blob)
```

#### 4.3 Identity Management

```
Checklist:
- [ ] Testar definição de roles
- [ ] Verificar workflow de user provisioning/deprovisioning
- [ ] Procurar por contas de teste
- [ ] Testar suspensão de conta
```

#### 4.4 Authentication

```
Checklist (combinado com A04):
- [ ] Credenciais padrão
- [ ] Weak password policy
- [ ] Session timeout
- [ ] Remember password vulnerability
- [ ] Browser cache weakness
- [ ] Weak authentication methods
- [ ] Weak security questions
- [ ] Weak password change/reset
- [ ] Alternative channels (SMS, email)
```

#### 4.5 Authorization

```
Checklist:
- [ ] Directory traversal
- [ ] File inclusion (LFI, RFI)
- [ ] Privilege escalation
- [ ] Insecure direct object references
- [ ] Bypass of user registration restrictions
```

#### 4.6 Session Management

```
Checklist:
- [ ] Session management schema
- [ ] Cookie attributes (Secure, HttpOnly, SameSite)
- [ ] Session fixation
- [ ] CSRF
- [ ] Cross Site Request Forgery prevention
- [ ] Session hijacking
- [ ] JSON Web Token (JWT)
```

#### 4.7 Input Validation

```
Checklist:
- [ ] SQL injection
- [ ] NoSQL injection
- [ ] Command injection
- [ ] LDAP injection
- [ ] XPath injection
- [ ] XXE injection
- [ ] XML injection
- [ ] HTML injection
- [ ] XSLT injection
- [ ] XQuery injection
```

#### 4.8 Error Handling

```
Checklist:
- [ ] Error code analysis
- [ ] Stack traces exponho
- [ ] Information disclosure
```

#### 4.9 Weak Cryptography

```
Checklist:
- [ ] Weak encryption
- [ ] Weak hashing
- [ ] Weak random number generation
```

#### 4.10 Business Logic

```
Checklist:
- [ ] Test business logic data validation
- [ ] Test ability to forge requests
- [ ] Test integrity checks
- [ ] Test for process timing
- [ ] Test number of times a function can be used
- [ ] Test circumvention of workflows
- [ ] Test defenses against application misuse
- [ ] Test authorization on business functions
```

#### 4.11 Client-Side

```
Checklist:
- [ ] DOM XSS
- [ ] HTML5 storage
- [ ] Cross-origin resource sharing (CORS)
- [ ] Clickjacking
- [ ] WebSockets
- [ ] Web Messaging
- [ ] Local storage
- [ ] Reverse Tabnabbing
```

***

### 5. ADVANCED TESTING - VULNERABILIDADES ESPECÍFICAS

#### 5.1 GraphQL

```
Checklist:
- [ ] Habilitar introspection query (__schema)
- [ ] Procurar mutations não autorizado
- [ ] Testar field-level authorization
- [ ] Verificar input validation em GraphQL
- [ ] Testar Denial of Service (deep nested queries)
- [ ] Procurar sensitive data exposure (fragments)
- [ ] Testar batch queries sem limite
- [ ] Verificar rate limiting
```

#### 5.2 WebSocket

```
Checklist:
- [ ] Testar origem validation
- [ ] Verificar autenticação em WebSocket upgrade
- [ ] Testar CSRF em WebSocket
- [ ] Procurar por data leakage em mensagens
- [ ] Verificar rate limiting em mensagens
- [ ] Testar injection via WebSocket
```

#### 5.3 HTTP Request Smuggling

```
Tipos:
CL.TE: Content-Length vs. Transfer-Encoding
TE.CL: Inverso

Checklist:
- [ ] Testar TE.CL vulnerability
- [ ] Testar CL.TE vulnerability
- [ ] Verificar response timing
- [ ] Testar cache poisoning via HTTP smuggling
- [ ] Procurar por session hijacking via smuggling
```

**Ferramentas**: Burp Scanner, http2smugl

#### 5.4 Cache Poisoning

```
Checklist:
- [ ] Procurar por parâmetros não cacheados (X-Forwarded-Host)
- [ ] Testar cache key injection
- [ ] Verificar header unkeyed (X-Original-URL, etc)
- [ ] Testar normalization flaws (case sensitivity)
- [ ] Procurar por reflected XSS cacheable
```

#### 5.5 Race Conditions

```
Checklist:
- [ ] Procurar por operações critical race-prone (transações, descontos)
- [ ] Testar requisições paralelas
- [ ] Verificar time-of-check vs. time-of-use (TOCTTOU)
- [ ] Testar double-spending
- [ ] Procurar por inventory races
```

#### 5.6 Clickjacking

```
Checklist:
- [ ] Verificar X-Frame-Options header
- [ ] Testar frame busting bypass
- [ ] Procurar por sensitive actions na interface
- [ ] Verificar Content-Security-Policy
```

#### 5.7 CSRF (Cross-Site Request Forgery)

```
Checklist:
- [ ] Procurar por state-changing requests
- [ ] Verificar CSRF tokens
- [ ] Testar double-submit cookies
- [ ] Verificar SameSite cookies
- [ ] Procurar por formulários sem token
- [ ] Testar CSRF em GET requests
- [ ] Verificar CORS + CSRF
- [ ] Procurar por CSRF em JSON POST
```

#### 5.8 XSS (Cross-Site Scripting)

```
Tipos:
Reflected: Input → Response (immediate)
Stored: Input → Database → Response (delayed)
DOM-based: Client-side JavaScript vulnerability

Payloads:
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg/onload=alert(1)>
'><script>alert(1)</script>
"><script>alert(1)</script>
<iframe src="javascript:alert(1)">

Checklist:
- [ ] Procurar por input fields (name, search, etc)
- [ ] Testar Reflected XSS
- [ ] Testar Stored XSS
- [ ] Testar DOM-based XSS
- [ ] Verificar Content Security Policy (CSP)
- [ ] Procurar por WAF bypass
- [ ] Testar encoding bypass (HTML, URL, JS)
- [ ] Verificar context (HTML, JavaScript, CSS, URL)
- [ ] Procurar por XSS em email/PDF
- [ ] Testar Blind XSS (via OOB channel)
```

**Ferramentas**: dalfox, XSSHunter, manual testing

#### 5.9 Content Injection

```
Checklist:
- [ ] HTML injection
- [ ] CSS injection (expresson() {IE})
- [ ] JavaScript injection (não-reflected)
- [ ] PDF injection
- [ ] Email injection
- [ ] Log injection
```

#### 5.10 SSTI (Server-Side Template Injection)

```
(Já coberto em A02, seção 2.2.6)
```

#### 5.11 File Upload Vulnerabilities

```
Checklist:
- [ ] Testar upload de arquivo executável (.php, .jsp, .asp)
- [ ] Bypass de extensão (.php.jpg, .php%20, .php::$DATA)
- [ ] Bypass de MIME type
- [ ] Procurar por path traversal em upload
- [ ] Testar XXE via arquivo XML
- [ ] Procurar por zip bomb / zip of death
- [ ] Testar polyglot files (PHP+JPG)
- [ ] Verificar validação de conteúdo (magic bytes)
- [ ] Procurar por race condition em upload
- [ ] Testar upload para write-protected directories
- [ ] Verificar se arquivo é executado ou servido
```

#### 5.12 LDAP Injection / Directory Traversal

```
(Já coberto em A02, seção 2.2.4)
```

#### 5.13 Subdomain Takeover

```
Checklist:
- [ ] Procurar por CNAMEs apontando para serviços extintos
- [ ] Verificar GitHub Pages, Heroku, Azure, etc.
- [ ] Procurar por broken subdomains
- [ ] Testar se pode registrar domínio
- [ ] Verificar histórico de DNS (Wayback Machine)
```

**Ferramentas**: subdomaintakeover, can-i-take-over-xyz

***

### 6. FERRAMENTAS E RECURSOS ESSENCIAIS

#### 6.1 Recon & OSINT Tools

```
Enumeração de Subdomains:
- Subfinder (fast, passive)
- Amass (comprehensive, OWASP)
- Sublist3r
- Findomain
- BBOT
- Knockpy (brute-force)
- Assetfinder

Procura de Endpoints:
- GAU (GetAllUrls) - procura em Wayback Machine
- Katana (web crawler)
- paramspider (parameter discovery)
- arjun (parameter discovery)

DNS & Network:
- DNSRecon
- dig, nslookup
- Massdns
- Nmap

Web Reconnaissance:
- Shodan / Censys / Netlas
- Certificate Transparency (crt.sh)
- WHOIS lookup
- Wappalyzer (tecnology stack)
```

#### 6.2 Scanning & Vulnerability Detection

```
Automated Scanners:
- Burp Suite Pro (atividade, passive/active scan)
- Nuclei (template-based, custom)
- ZAP (OWASP, gratuito)
- Invicti
- Checkmarx
- Sonarqube (SAST)

Specific Checkers:
- sqlmap (SQL injection)
- paramspider (parameter discovery)
- gf (grep finder - payloads filtering)
- ffuf (fuzzing)
```

#### 6.3 Manual Testing Tools

```
Proxy & Interception:
- Burp Suite Community/Pro (essential)
- OWASP ZAP
- Fiddler

Payload Generation:
- PayloadsAllTheThings (GitHub repo - 1000+ payloads)
- SecLists (wordlists)
- fuxploider (CMS fuzzer)

API Testing:
- Postman
- Insomnia
- curl
- GraphQL Playground
```

#### 6.4 Specialized Tools

```
JWT:
- jwt.io
- jwt_tool.py
- Burp JWT extension

Deserialization:
- ysoserial (Java gadget chains)
- marshalsec (Java)

XXE / XML:
- XXE Injector

Crypto:
- hashcat
- John the Ripper

Networking:
- nc (netcat)
- tcpdump
- Wireshark
```

#### 6.5 Wordlists & Payloads

```
Repositórios:
- PayloadsAllTheThings (GitHub)
- SecLists (dirb, fuzzing, etc)
- fuxploider

Custom Wordlists:
- Construir baseado em target
- Aplicações específicas
- Padrões de naming observados
```

***

### 7. CHECKLIST FINAL - ANTES DE SUBMETER

```
VALIDAÇÃO DE VULNERABILIDADE:
- [ ] Vulnerabilidade é reproduzível?
- [ ] Funcionado em versão atual?
- [ ] Impacto é claro?
- [ ] Prova de conceito é limpa e não destrutiva?

PREPARAÇÃO DO RELATÓRIO:
- [ ] Título descritivo
- [ ] Descrição técnica clara
- [ ] Steps para reprodução (passo a passo)
- [ ] Impacto comercial/técnico
- [ ] Prova de conceito (screenshots, logs, payloads)
- [ ] Severidade apropriada (CVSS)
- [ ] Correção sugerida

VERIFICAÇÃO FINAL:
- [ ] Não divulgou dados sensíveis reais
- [ ] Não danificou nada
- [ ] Não violou escopo do programa
- [ ] Seguiu política de divulgação
- [ ] Escrita profissional
- [ ] Sem divulgação prévia
```

***

### 8. REFERÊNCIAS E RESOURCES

#### Frameworks & Metodologias

* OWASP Top 10 2025
* OWASP API Security Top 10 2023
* OWASP Web Security Testing Guide (WSTG)
* OWASP Testing Cheat Sheet
* PTES (Penetration Testing Execution Standard)
* NIST SP 800-115
* Microsoft Threat Modeling
* CWE Top 25 (MITRE)

#### Comunidades & Plataformas

* HackerOne
* Intigriti
* Bugcrowd
* YesWeHack
* Hackerone Report 2025

#### Repos Essenciais

* PayloadsAllTheThings (swisskyrepo/GitHub)
* SecLists (danielmiessler/GitHub)
* Web Security Academy (PortSwigger)
* OWASP Projects

#### Leitura Recomendada

* "The Web Application Hacker's Handbook"
* PortSwigger Web Security Academy
* HackTheBox & TryHackMe labs
* OWASP Testing Guide
* Bug Bounty Playbook

***

**Nota Final**: Este checklist deve ser adaptado para cada aplicação específica. Nem todas as vulnerabilidades serão aplicáveis a toda aplicação. Comece com threat modeling para identificar riscos específicos e priorize seus testes conforme o risco e impacto potencial.

## Golden Window Strategy

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

