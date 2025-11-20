---
description: RustScan - High-Speed Port Scanner
icon: rust
---

# Rustscan

RustScan é a ferramenta de escolha quando a velocidade na varredura de todas as 65535 portas é prioritária, atuando como um "pré-scanner" extremamente rápido para alimentar o Nmap com as portas abertas.

#### Checklist Conceitual (RustScan)

1. Priorizar Velocidade: Usar o _ulimit_ (ou valor padrão alto) para maximizar as conexões TCP assíncronas.
2. Integração com Nmap: Encadear o RustScan diretamente ao Nmap para varredura de serviços/scripts.
3. Identificar Alvos: Varredura rápida de todas as portas (1-65535) para cobertura máxima.
4. Evasão (Rate Limit): Ajustar o _timeout_ para evitar _rate limiting_ simples.

***

```json
{
  "comandos": [
    "# 1. Scan Completo (65535 portas) e Output para Nmap (Recomendado)",
    "rustscan -a 10.10.1.5 --ulimit 5000 -- -sV -sC -oA full_scan",
    "# 2. Scan Completo e Rápido (Somente portas abertas)",
    "rustscan -a 10.10.1.5 --ulimit 5000",
    "# 3. Scan Focado em Portas Específicas e Comandos Nmap Customizados",
    "rustscan -a 10.10.1.5 -p 22,80,443,8080 -- -A --script http-enum",
    "# 4. Scan de Range de IPs/Sub-redes (Host Discovery)",
    "rustscan -a 10.10.1.0/24",
    "# 5. Ajuste do Timeout (para lidar com redes lentas ou instáveis)",
    "rustscan -a 10.10.1.5 --timeout 500ms",
    "# 6. Carregar Alvos de um Arquivo (Ex: lista de IPs para Red Team)",
    "rustscan -a targets.txt -- -sV"
  ],
  "workflow": "1. **Inicialização:** Usar RustScan com `--ulimit 5000` (ou mais, dependendo do sistema) e `--a` para obter rapidamente todas as portas abertas (1-65535).\n2. **Pipeline:** **Encadear** o RustScan ao Nmap (usando `-- --`) para que o Nmap rode apenas nas portas que o RustScan identificou como abertas. Isso economiza tempo crítico.\n3. **Detalhamento de Serviços:** O Nmap executa detecção de versão (`-sV`) e scripts padrões (`-sC`) nas portas abertas (ex: 80, 443, 3389, 1433).\n4. **Análise:** Salvar o resultado (XML ou Grepable) do Nmap para ferramentas de pós-processamento (ex: **EyeWitness** para web ou **GoBuster/DirBuster** para enumeração de diretórios).",
  "taticas_avancadas": [
    "**Scan Sequencial/Lento:** Em ambientes sensíveis a *rate limiting* ou *honeypots*, use a flag `--nmap-cli '-T2'` para diminuir drasticamente a velocidade do Nmap, mas mantendo a velocidade inicial do RustScan (que só identifica a porta aberta).",
    "**Varredura Rápida de 'Alvos Quentes':** Utilizar a flag `--top` para varrer apenas as N portas mais comuns. Ex: `rustscan -a $IP --top 1000` antes do scan completo.",
    "**Output para Nmap com Ação Específica:** Usar `grepable output` (`-oG`) do Nmap no pipeline para automatizar ações subsequentes. Ex: `rustscan ... | nmap ... -oG - | grep open | cut -d' ' -f2 | xargs -I {} -P 10 bash -c 'nikto -h {}'`",
    "**Tunelamento:** Utilizar RustScan por trás de um túnel SSH ou proxy (ex: **ProxyChains**), embora a velocidade e a natureza assíncrona do RustScan possam causar *timeouts* e resultados inconsistentes. **Alternativa:** Usar `nmap` diretamente via *ProxyChains* apenas nas portas identificadas pelo RustScan."
  ],
  "praticas_seguras": [
    "**Controle de Ulimit:** Mantenha o `--ulimit` em um valor razoável (3000-5000) para evitar o esgotamento dos descritores de arquivo no sistema operacional do atacante.",
    "**Use Nmap para Scripts:** Evite tentar varredura profunda com RustScan. Ele deve ser usado **apenas** para identificar portas. O trabalho de detecção de serviço, versão e vulnerabilidade (que pode ser ruidoso) deve ser delegado ao Nmap (`-sV -sC`).",
    "**Timeout (Real-World Engagements):** Use um *timeout* maior (e.g., `--timeout 1000ms`) em ambientes de produção com firewalls complexos ou redes lentas para evitar falsos negativos (portas abertas que são reportadas como fechadas devido a *timeout* rápido).",
    "**Varredura Mínima (Targeted Scanning):** Em Red Team, use RustScan apenas para varrer portas específicas que são relevantes ao vetor de intrusão, e não todas as 65535, para minimizar o *footprint*."
  ],
  "tradeoffs_performance": "RustScan é o scanner de portas **mais rápido** em muitos testes, ideal para *initial sweep* (varredura inicial). \n\n**Trade-offs/Limitações:**\n1. **Foco em TCP:** Embora suporte UDP, o RustScan é otimizado para a varredura TCP e não é recomendado para varredura UDP profunda (onde o Nmap ou ZMap são mais adequados).\n2. **Menos Stealth:** A alta velocidade (conexões em massa) é facilmente detectada por *firewalls* e IDS/IPS configurados para monitorar *burst scanning*.\n3. **Falsos Positivos/Negativos:** O modelo assíncrono e de *timeout* agressivo pode, ocasionalmente, retornar falsos positivos em redes ruidosas ou falsos negativos em *hosts* com *rate limiting* complexo. **Sempre valide o resultado com o Nmap**.\n\n**Quando Usar:** Ideal no início de um *engagement* para economizar tempo, alimentando o Nmap em menos de 10 segundos.",
  "fontes_referencia": [
    {
      "titulo": "RustScan Documentation (GitHub) - Configuração e Flags",
      "url": "https://github.com/RustScan/RustScan"
    },
    {
      "titulo": "HackTricks - Port Scanning (Comparativo com Nmap e Outros)",
      "url": "https://book.hacktricks.xyz/network-services-pentesting/port-scanning"
    },
    {
      "titulo": "Guia de Performance (The Hacker News, Artigos sobre Velocity Scanning)",
      "url": "https://thehackernews.com/tag/port-scanning"
    },
    {
      "titulo": "PayloadAllTheThings - Network Scanning (Estratégias Avançadas)",
      "url": "https://swisskyrepo.github.io/PayloadsAllTheThings/Methodology%20and%20Resources/Network%20Scanning/"
    }
  ]
}
```
