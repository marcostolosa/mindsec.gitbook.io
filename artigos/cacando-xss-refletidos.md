---
description: '[pt-br] Hunting for reflected XSS vulnerabilities: A complete guide'
---

# Caçando XSS refletidos

## O que é Cross-site scripting (XSS)

A vulnerabilidade de Cross-site scripting (XSS) é uma falha de injeção que permite ao atacante injetar scripts maliciosos em páginas web visualizadas por outros utilizadores. Isso funciona explorando validação de entrada insuficiente e falta de codificação de qualquer "reflexão" desse input do utilizador, permitindo que o atacante insira código HTML ou JavaScript que será executado no navegador da vítima ao visitar a página vulnerável.

Dependendo do componente vulnerável, o XSS pode ocorrer tanto no lado do cliente quanto no lado servidor. O XSS do lado cliente ocorre inteiramente no navegador da vítima, e permite ao atacante assumir a sessão da vítima. O XSS do lado servidor (também referido como “blind XSS”) ocorre quando o componente vulnerável utiliza seu input não ­sanitizado e avalia-o no lado servidor, por exemplo via um navegador headless. Isso permite ao atacante usar código JavaScript arbitrário para iniciar pedidos em nome do servidor, e em casos graves até ler ficheiros locais (dependendo da configuração do navegador headless).

Agora vamos dar uma olhada nos diferentes tipos de vulnerabilidades XSS.

### **XSS refletido**

O XSS refletido (ou por vezes referido como “reflective XSS”) ocorre quando um input malicioso do utilizador é inserido através de uma propriedade do pedido (como o caminho da URL, fragmento, parâmetro query/body, ou cabeçalho HTTP) e é imediatamente refletido de volta ao utilizador sem sanitização adequada.\
O servidor processa o input do utilizador e inclui-o na resposta HTTP sem qualquer codificação, causando que o navegador da vítima execute o script do atacante quando ela clicar num link especialmente criado.

### **XSS armazenado**

O XSS armazenado (ou persistent XSS) acontece quando um script malicioso do atacante fica guardado na base de dados alvo, no sistema de ficheiros, ou qualquer outro serviço de armazenamento (como AWS S3). Quando outros utilizadores mais tarde recuperarem esse dado armazenado (por exemplo visualizando uma secção de comentários, perfil público ou postagem de fórum), o script malicioso executa-se no browser deles.

O XSS armazenado é particularmente perigoso porque pode afetar múltiplas vítimas sem exigir que elas cliquem num link específico, ao contrário do XSS refletido.

#### **XSS baseado em DOM**

O XSS baseado em DOM ocorre quando o código JavaScript inseguro processa dados controláveis pelo utilizador (provenientes de uma fonte DOM) e os passa para um “sink” do DOM. Isso permite ao atacante conceber payloads JavaScript que serão avaliados pela aplicação vulnerável.

O XSS DOM é mais difícil de detectar porque o input malicioso não aparece imediatamente refletido na resposta HTTP, mas sim passa por processamento no DOM. A identificação e exploração de vulnerabilidades XSS baseada em DOM serão tratadas num próximo artigo.

Ao longo deste artigo vamos focar exclusivamente no XSS refletido (ou reflective) e no XSS armazenado (persistent), já que ambos partilham características semelhantes.

> E o self-XSS?&#x20;
>
> O self-XSS acontece quando um atacante convence a vítima a executar JavaScript malicioso no seu próprio browser, tipicamente pedindo que cole código no console do developer ou num campo de texto num website legítimo ao qual ela tem acesso (por exemplo o campo de endereço no seu perfil).&#x20;
>
> Embora isto tecnicamente cause execução de script, a maioria dos programas de bug-bounty e investigadores de segurança não consideram self-XSS como risco de segurança porque exige que a vítima ativamente se ataque a si própria — o que quebra o princípio fundamental de que vulnerabilidades devem ser exploráveis sem engenharia social extensa.\
> Observe que existem casos onde self-XSS pode ser encadeado e escalado, mas isso será tratado mais extensivamente num artigo futuro.

## Metodologia

Se você é principiante, esta parte é essencial. Pode ajuda-lo a poupar horas ao determinar se encontrou uma vulnerabilidade XSS ou está a lidar com uma simples injeção de conteúdo. Vamos passar por esta metodologia de 3 passos para ajudar a identificar uma vulnerabilidade XSS refletida ou armazenada.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### **Passo 1: Reflexão**

O primeiro passo é identificar onde o seu input é refletido na resposta da aplicação. Insira uma string única (como `1337test`) em vários campos de input, parâmetros de URL, cabeçalhos ou qualquer outro ponto de dados controlável pelo utilizador.

Em seguida, procure essa string na resposta HTTP. Isto ajuda-lo a mapear todos os pontos de reflexão e entender onde o seu input acaba — o que é crucial para determinar se a exploração é possível e que tipo de injeção se está a tratar.

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

> Dica: Enumerar parâmetros “hidden” pode ajuda-lo a descobrir todos os tipos de vulnerabilidades de injeção, incluindo XSS!

* https://www.intigriti.com/researchers/blog/hacking-tools/finding-hidden-input-parameters

### **Passo 2: Injeção**

Uma vez encontrado um ponto de reflexão, teste se consegue “sair” do contexto atual injetando uma simples tag HTML como `<s>1337test` e observar como a aplicação a lida.

Para reflexões em contexto JavaScript (qualquer coisa dentro de `<script>`), a string de injeção será ligeiramente diferente — será discutido mais em detalhe abaixo.

#### **Entrada refletida sem injeção**

Se o seu input for refletido mas não injetável (por exemplo foi codificado em HTML), então provavelmente está protegido — neste caso, XSS muitas vezes não é possível. No entanto, atenção: há “caveats”, por exemplo aplicações que se comportam diferente consoante user-agent (mobile vs desktop) ou quando se injetam caracteres especiais como null bytes, CR/LF, etc.

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

#### **Entrada refletida com injeção**

Se o seu input for refletido **e** injetável — ou seja, sem filtro ou codificação — então tem uma forte indicação de que XSS é possível. Tudo o que falta é injetar uma tag maliciosa ou event handler para transformar a simples injeção HTML num XSS de facto.

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

### **Passo 3: Payload (prova de conceito)**

Agora é altura de conceber uma PoC (proof of concept) que execute JavaScript no navegador da vítima. Para tal, o seu payload depende de dois fatores:

1. O contexto em que o seu input é refletido
2. Quaisquer filtros existentes que o impeçam de injetar payloads XSS maliciosos

#### **Contexto genérico (HTML)**

Quando o seu input é diretamente refletido no corpo HTML sem estar envolvido por nenhuma tag ou atributo específico, tem a maior flexibilidade para exploração.

Comece com payloads simples:

```
<script>alert(1)</script>
```

ou:

```
<img src=x onerror=alert(1)>
```

Estes payloads funcionam sem precisar de sair de nenhuma tag. Este é o cenário mais fácil para iniciantes, já que você pode usar virtualmente qualquer tag HTML que suporte execução de JavaScript, incluindo `<svg>`, `<iframe>`, `<object>`, ou atributos de event handler em tags self-closing.

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

#### **Atributo HTML (HTML inline)**

Quando o XSS reflected aparece dentro de um atributo HTML (ex: `<input value="REFLECTION">` ou `<form action="/path?param=REFLECTION">`), vai precisar de sair do contexto do atributo antes de injetar o payload.

Pode fechar o atributo com uma aspa (`“` ou `‘`), fechar a tag com `>`, e depois injetar uma nova tag maliciosa como `"><script>alert(1)</script>`, ou permanecer na mesma tag adicionando um event handler como `" onload=alert(1) x="`.

A chave é perceber que tipo de aspas (simples ou duplas) são usadas para envolver o atributo e se a aplicação filtra outros caracteres que poderiam impedir-nos de escapar o contexto.

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

#### **Bloco JavaScript**

Quando o seu input é refletido dentro de tags `<script>`, tipicamente como parte de uma atribuição de variável, por exemplo, `var data = "REFLECTION";` ou `var config = {"key":"REFLECTION"};`, você vai precisar sair da sintaxe JavaScript para conseguir executar seu código.

Por exemplo para strings comuns, escape com `"; alert(document.domain); //` para fechar a string, executar o seu código, e comentar o resto.

Para literais de template (backticks) pode usar `${alert(document.domain)}` para executar código diretamente dentro do template sem quebrar a sintaxe.

A parte complicada é garantir que o JavaScript vai permanecer sintaticamente válido após a sua injeção — portanto preste atenção a parênteses, aspas ou chaves não fechadas que podem causar erro e impedir a execução.

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

## Exploração de XSS

Já cobremos o que são vulnerabilidades XSS e como podemos metodicamente procurar possíveis pontos de injeção. Agora é altura de pôr as suas novas habilidades à prova. Vamos dar uma olhada em alguns exemplos reais para ajudar a conceber os seus payloads.

### **XSS sem qualquer filtragem**

Em casos raros, encontrará cenários onde não existe nenhuma filtragem ou validação para prevenir vulnerabilidades XSS. Esses normalmente resultam de negligências de desenvolvedores que se esquecem de sanitizar o input do utilizador.

Nestes casos, pode usar praticamente qualquer payload para provar a existência do XSS.

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

### **XSS em HTML inline (atributos)**

Outro cenário relativamente trivial é onde o input é refletido no valor de um atributo HTML. Aqui pode ou:

* Sair do atributo e injetar um event handler para executar código JS arbitrário
* Ou sair do atributo, fechar a tag e abrir uma nova tag para executar JS arbitrário

Dependendo do contexto, normalmente é mais fácil injetar um event handler. Noutras situações pode ser a única solução, caso os caracteres usados para abrir/fechar tags estejam explicitamente filtrados.

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

### **XSS em contexto JavaScript**

À medida que as aplicações se tornam mais complexas, os desenvolvedores tendem a introduzir mais funcionalidades, e também mais pontos de falha, incluindo vulnerabilidades XSS. Embora este contexto ocorra com menor frequência, algumas vezes o developer reflete input não-sanitizado directamente no contexto JavaScript. Muitas vezes sem perceber as consequências. Aqui podemos usar esta oportunidade para escapar do contexto e injetar código arbitrário sem usar HTML. Consoante o ponto de reflexão, normalmente vai precisar de:

* Escapar o contexto atual (frequentemente uma string de variável ou parâmetro de função)
* Injetar o seu payload
* Fechar o payload para que a sintaxe fique correta

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

### **XSS dentro do campo text area**

Browsers, com algumas tags HTML como `<textarea>`, recusarão renderizar o seu valor para preservar o conteúdo. Se for principiante, poderá inicialmente julgar este cenário como “não explorável”. No entanto, se o seu input for refletido dentro de qualquer uma destas tags, terá de fechar a tag antes de injetar o seu payload.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

### **XSS com HTML & atributos limitados**

Um cenário muito comum que irá encontrar é onde o input é limitado, filtrado ou bloqueado sempre que um padrão específico é encontrado — este é de longe um dos métodos de filtragem mais usados por desenvolvedores.

Felizmente para nós, em muitos casos basta fazer o fuzzing (testar repetidamente) por quaisquer tags HTML permitidas, event handlers e caracteres para nos ajudar a contornar o filtro ou o WAF (Web Application Firewall).

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

> Dica: A PortSwigger Research Academy disponibiliza uma cheat-sheet com todas as tags HTML e atributos disponíveis. Podemos usar essa lista para praticamente enumerar todas as tags aceitas e event handlers, e depois combinar as peças do puzzle injetando uma tag permitida + atributo para criar o payload!

* https://portswigger.net/web-security/cross-site-scripting/cheat-sheet

## Conclusão

O fator chave deste artigo é **identificar o contexto** do seu ponto de injeção e compreender como o alvo reage a caracteres maliciosos. Sempre que estiver a testar alvos que implementam regras de filtragem estritas, é recomendado que leve tempo para **testes manuais** nos pontos de reflexão. Algumas ferramentas podem falhar em detectar estes cenários e por isso **não podem ser totalmente confiáveis**.

Então, acaba de aprender algo novo sobre como caçar vulnerabilidades XSS refletidas… Agora é hora de pôr as suas habilidades em ação! Pode começar por praticar em labs vulneráveis e CTFs ou… passar por 70+ programas públicos de bug-bounty na Intigriti — e quem sabe ganhar uma recompensa na sua próxima submissão!

***

## Insights & Dicas

* Nunca dependa só de scanners automáticos para XSS refletido: muitos vão falhar em contextos complexos (ex: JS block, atributos alterados dinamicamente). O artigo já aponta isso: faça **manual review**.
* Quando trabalhar em bug-bounty, **anote o contexto EXATO** do XSS reflection (HTML body, atributo, JS string, template literal). Isso ajuda não só a conceber o payload correto como também a escrever um relatório mais eficaz para o programa.
* Use payloads “mínimos”, depois evolua para bypass filters — comece com `<s>test`, `<img src=x onerror=alert(1)>`, depois parta para contextos JS como `";alert(1);//`. Assim consegue mapear os filtros mais rapidamente.
* Fuzzing + “allowed tags list” = um padrão vencedor: use cheat-sheets (\~PortSwigger) para enumerar tags/atributos permitidos, e depois injete sistematicamente.
* Lembre-se: canais menos óbvios de input/reflection (headers, referrer, fragmento de URL, path) são bons alvos. Muitos alvos focam em parâmetros query/body, mas cabeçalhos/paths podem escapar da revisão...
* Documente bem cada passo no relatório: identificação do ponto de reflection, contexto, payload usado, evidência da execução, impacto (session hijack, internal resource access, etc.). Isso aumenta chances da vulnerabilidade ser aceita e apreciada.
* Mantenha-se atualizado: filtros, frameworks modernas e WAFs evoluem; o que funcionava há 2-3 anos pode já não funcionar. Invista tempo em labs recentes e compartilhe conhecimento na comunidade.
* Como hacker jaded: **priorize profundidade sobre volume**. Encontrar “o” XSS tricky em contexto JS ou template literal vale mais que 10 triviais que já estão saturados nas plataformas.



#### Referencias:

* [https://www.intigriti.com/researchers/blog/hacking-tools/hunting-for-reflected-xss-vulnerabilities](https://www.intigriti.com/researchers/blog/hacking-tools/hunting-for-reflected-xss-vulnerabilities)

