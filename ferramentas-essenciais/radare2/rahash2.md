# rahash2

### Visão geral

`rahash2` é o canivete suíço de **hash**, **entropia**, **(de|en)cifra** e **transcodificações** da suíte radare2. Funciona muito bem em **pipelines**.

> Dica: `-` como argumento de arquivo significa **STDIN**.

### Hashes rápidos

```bash
# Hash de arquivo (sha256)
rahash2 -a sha256 malware01.ps1

# Vários algoritmos de uma vez
rahash2 -a md5,sha1,sha256 malware01.ps1

# Hash via STDIN (bom pra pipes)
echo -n "https://mindsecurity.org" | rahash2 -a md5 -
```

### Entropia (detectar cifrado/compactado)

```bash
rahash2 -a entropy data.enc
# ~7.9–8.0 bits/byte ⇒ provável ciphertext/arquivo comprimido
```

### Base64 (encode/decode)

```bash
# Decode Base64 -> binário
echo -n "SGF6ZQ==" | rahash2 -D base64 - > out.bin

# Encode Base64 a partir de arquivo
cat out.bin | rahash2 -E base64 -
```

### AES-CBC (decrypt) com Key/IV **binários**

```bash
# Key: 32 bytes (AES-256) em arquivo Key
# IV : 16 bytes em arquivo IV
# -S @Key => lê chave do arquivo
# -I HEX  => IV em hexstring; convertemos bytes->hex com rax2 -S
rahash2 -D aes-cbc -S @Key -I $(rax2 -S < IV) data.enc > data.dec
```

### HMAC

```bash
# HMAC-SHA256 com chave em arquivo
rahash2 -a hmac-sha256 -S @Key mensagem.bin
```

### Boas práticas

* Use `-` para STDIN em pipelines.
* Combine com `file`, `xxd`, `strings` para triagem.
* Confirme modo de cifra (CBC/CTR/GCM) antes de tentar decriptar.
