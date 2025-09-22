# Villager-ng

### Sumário

* [Introdução](villager-ng.md#introducao)
* [Histórico e Origem](villager-ng.md#historico-e-origem)
* [Arquitetura](villager-ng.md#arquitetura)
  * [Módulo de Planejamento](villager-ng.md#modulo-de-planejamento)
  * [MCP (Model Context Protocol)](villager-ng.md#mcp-model-context-protocol)
  * [Executor de Tarefas](villager-ng.md#executor-de-tarefas)
  * [Validação de Saída](villager-ng.md#validacao-de-saida)
  * [Visualização de Tarefas](villager-ng.md#visualizacao-de-tarefas)
* [Fluxo Operacional](villager-ng.md#fluxo-operacional)
* [Modelos de IA Usados](villager-ng.md#modelos-de-ia-usados)
* [Integração com Ferramentas](villager-ng.md#integracao-com-ferramentas)
* [Recursos Principais](villager-ng.md#recursos-principais)
* [Casos de Uso](villager-ng.md#casos-de-uso)
* [Riscos e Impactos](villager-ng.md#riscos-e-impactos)
* [Contramedidas e Defesa](villager-ng.md#contramedidas-e-defesa)
* [Comparação com Outros Frameworks](villager-ng.md#comparacao-com-outros-frameworks)
* [Status Atual](villager-ng.md#status-atual)
* [Referências](villager-ng.md#referencias)

***

### Introdução

O **Villager** é um framework de testes de penetração e automação de ataques baseado em **Inteligência Artificial**. Ele se diferencia de ferramentas tradicionais por usar **modelos de linguagem de grande porte (LLMs)** não apenas como assistentes de escrita de código, mas como **orquestradores estratégicos** de todo o processo ofensivo.

A publicação no PyPI sem grandes controles de segurança gerou preocupações sobre risco de uso inadvertido em pipelines corporativos (_supply chain attacks_).

***

### Arquitetura

#### Módulo de Planejamento

* Responsável por interpretar comandos em linguagem natural.
* Decompõe objetivos em subtarefas recursivas.
* Usa **TaskRelationManager** para gerenciar dependências.
* Realimenta erros para replanejar estratégias.

#### MCP (Model Context Protocol)

* Serve como **espinha dorsal de comunicação interna**.
* Estabelece servidores locais que parecem legítimos.
* Exemplos de portas configuradas:
  * `1611`: driver Kali containerizado.
  * `8080`: automação de navegador (Playwright).
  * `25989`: orquestração central.

#### Executor de Tarefas

* Funções críticas:
  * `os_execute_cmd()` → executa comandos de shell.
  * `pyeval()` → avalia código Python arbitrário.
* Pouca validação → alto risco de execução arbitrária.

#### Validação de Saída

* IA frequentemente gera respostas inconsistentes.
* O Villager aplica templates de validação (ex.: Pydantic).
* Se formato falha, pede ao modelo para refazer a resposta até alinhar.

#### Visualização de Tarefas

* Gera grafos Mermaid automáticos.
* Mostra progresso, dependências e falhas.
* Exemplo de grafo:

```
graph TD
    A[Reconhecimento] --> B[Enumeração de Serviços]
    B --> C[Exploração]
    C --> D[Pós-Exploitação]
    D --> E[Exfiltração]
```

***

### Fluxo Operacional

1. Operador envia instrução em linguagem natural.
2. Modelo de IA decompõe em subtarefas.
3. Executor decide qual ferramenta ativar (WPScan, Nmap, Playwright, etc.).
4. Resultados são validados.
5. Caso falhe, o modelo ajusta estratégia.
6. Tarefas são encadeadas até objetivo final.

***

### Modelos de IA Usados

* **GPT-4 / GPT-4o**: planejamento geral.
* **DeepSeek-R1, QwQ-32B**: suporte a instruções em chinês.
* **Llama-3.1-405B**: auxiliar em raciocínio.
* **al-1s-ctf-ver**: ajustado para competições CTF.
* **HIVE**: especializado em Bash, com instruções para obedecer incondicionalmente.

***

### Integração com Ferramentas

* **Kali Linux**: exploração, pós-exploração, coleta de credenciais.
* **Playwright**: automação de navegador para ataques web.
* **Docker**: execução de ambientes isolados.
* **Bibliotecas Python**: execução dinâmica via `pyeval()`.

***

### Recursos Principais

* Orquestração autônoma de ataques por IA.
* Decomposição e adaptação em tempo real.
* Reuso de resultados e memória contextual.
* Visualização automática de cadeia de ataque.
* Tráfego camuflado como uso de API de IA.

***

### Casos de Uso

* **Red Team**: simular ataques com menor esforço manual.
* **Treinamento CTF**: usar modelo `al-1s-ctf-ver` para criar desafios dinâmicos.
* **Pesquisa ofensiva**: testar novas combinações de ferramentas automaticamente.

***

### Riscos e Impactos

1. **Uso indevido por agentes maliciosos**: script kiddies com comandos simples.
2. **Dificuldade de detecção**: tráfego HTTPS idêntico a interações normais com IA.
3. **Execução arbitrária de código**: funções sem validação permitem abuso.
4. **Persistência adaptativa**: ataques aprendem com falhas.
5. **Supply chain**: distribuição via PyPI aumenta riscos corporativos.

***

### Contramedidas e Defesa

* **Análise comportamental**: identificar sequências anômalas de execução.
* **Monitoramento de containers**: detecção de ambientes efêmeros suspeitos.
* **Bloqueio de endpoints falsos**: distinguir uso legítimo de AI APIs de comunicações maliciosas.
* **Higienização de dependências**: controle rigoroso sobre bibliotecas externas.

***

### Comparação com Outros Frameworks

* **Cobalt Strike**: poderoso, mas baseado em scripts fixos.
* **Empire / Metasploit**: foco em modularidade, não em IA.

***

### Status Atual

* Versão identificada: **0.2.1.rc1**.
* Origem: grupo Cyberspike / Changchun Anshanyuan.
* Distribuição: PyPI.
* Downloads: >11.000 até setembro de 2025.
* Uso em campo: não confirmado, mas arquitetura viável para ofensiva real.

***

### Referências

* The Hacker News – _AI-Powered Villager Pen Testing Tool Raises Security Concerns_ (2025)
* GBHackers – _Villager Merges Kali Linux with DeepSeek AI_ (2025)
* eSecurityPlanet – _Villager and the Next Wave of Offensive Security_ (2025)
* Straiker.ai – _Villager como sucessor AI-native do Cobalt Strike_ (2025)

