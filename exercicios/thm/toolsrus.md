---
description: https://tryhackme.com/room/toolsrus
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# ToolsRus

## `Hackeando Servidor com Tomcat Vulnerável – Do Enum ao Root`

1. **Introdução**
   * Objetivo: prática e aprendizado de hacking/pentest
   * Contexto do laboratório: room do tryhackme.com
   * Ferramentas: ffuf/gobuster, nmap, hydra, nikto, metasploit
2. **Enumeração**
   * Nmap (comando + saída explicada)
   * Dirbuster/FFUF (wordlists e resultados)
   * Nikto (versão e vulnerabilidades)
3. **Identificação do Tomcat Vulnerável**
   * Versão encontrada
   * CVE relacionado
   * Referências
4. **Acesso Inicial**
   * Hydra (comando, wordlist, resultado)
   * Painel Tomcat Manager
5. **Exploração**
   * Metasploit (módulo usado)
   * Geração de WAR reverso
   * Conexão via shell
6. **Escalada de Privilégios**
   * Enumeração interna (`linpeas` ou manual)
   * Exploit de root
7. **Mitigações**
   * Atualização do Tomcat
   * Configuração de senhas fortes
   * Restrição de acesso ao painel
8. **Conclusão**
   * Lições aprendidas
   * Próximos labs recomendados
