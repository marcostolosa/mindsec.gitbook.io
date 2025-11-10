---
description: Cross-site Scripting 3xposed!
---

# XSS

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
