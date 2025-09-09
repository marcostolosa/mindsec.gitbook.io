---
icon: dragon
---

# Ghidra

## Cheatsheet Ghidra&#x20;

***

### Atalhos de Teclado Essenciais

* **n** → renomear símbolo/variável/função
* **l** → rotular endereço
* **;** → comentar linha
* **Shift+E** → mudar tipo de variável (ex: `int` → `char*`)
* **Ctrl+Shift+G** → ir para endereço
* **G** → “go to” (vai direto a um endereço/label)
* **Ctrl+Shift+F** → buscar string/instrução/símbolo
* **Ctrl+Alt+H** → histórico de navegação (voltar/avançar)
* **Ctrl+Shift+P** → buscar instrução por padrão (regex ou assembly)

***

### Janelas e Views

* **Listing** → visão assembly
* **Decompile** → decompilação em C-like
* **Symbol Tree** → funções, variáveis globais, labels
* **Data Type Manager** → structs, enums, typedefs
* **Program Trees** → mapeamento das seções (ex: `.text`, `.data`, `.rdata`)
* **Function Graph** → grafo de fluxo da função (CFG)
* **Byte Viewer** → hex dump estilo binário cru
* **Symbol References** → cross-references (XREFs)

***

### Navegação e XREFs

* **X** → ver referências para o endereço/função
* **Ctrl+Shift+X** → todas as XREFs (callers + data)
* **Shift+U** → desfazer análise automática
* **Ctrl+Shift+Y** → mostrar funções que chamam a função atual
* **Ctrl+Shift+U** → mostrar funções chamadas pela função atual

***

### Análise e Edição

* **Definir função manual**: selecione instruções → **Right Click → Create Function**
* **Definir variável global**: selecione endereço → **Right Click → Data → Define → DWORD/PTR/etc.**
* **Estruturas**:
  * Crie no **Data Type Manager** (`Right Click → New Structure`)
  * Aplique no decompilado (ex: mudar `local_10h` → `MyStruct*`)
* **Patch Code**:
  * **Right Click → Patch Instruction** → sobrescreve opcode
  * **Right Click → Patch Bytes** → edita hex diretamente
* **FuncSig**: renomear e re-tipar parâmetros para reconstruir protótipo da função

***

### Analisando Malware

* **Strings**: `Window → Defined Strings` (procure comandos, paths, chaves)
* **Entrypoint**: `Analysis → Program Information` → `_start` / `mainCRTStartup`
* **Imports**: `Window → Symbol Tree → Imports` (funções da API chamadas)
* **Exports**: `Symbol Tree → Exports` (útil em DLLs)
* **Function Call Graph**: mapeia fluxo → útil pra achar lógica maliciosa
* **Data Flow**: rastreia valores → **Right Click → Show References to/From**
* **Decompiler**: altera tipos (ajuda a identificar APIs obscuras ou wrappers)
* **Control Flow Graph**: visualizar obfuscações/loops anômalos

***

### Exploit Dev / Vulnerabilidades

* **Buffers**: procure por funções inseguras (`strcpy`, `sprintf`, `gets`)
* **Cross-Refs**: veja quem chama essas funções
* **Stack Variables**: na decompilação, veja `char local_40[32]` → candidato a buffer overflow
* **Funcções custom de parsing**: use `XREFs` para achar entrada do usuário → fluxo até vulnerabilidade
* **Syscalls diretos**: em malware/exploits, Ghidra resolve `int 0x80` / `syscall` com IDs (bom para RE em Linux/Windows)

***

### Scripts e Scripting API

Ghidra tem **GhidraScript** (Java ou Python via Jython).

#### Exemplos rápidos:

```python
# Listar todas funções
for f in getFunctionManager().getFunctions(True):
    print(f.getName())

# Dump de strings
from ghidra.program.model.address import AddressSet
strings = currentProgram.getListing().getDefinedData(True)
for s in strings:
    if s.getDataType().getName() == "string":
        print(s)
```

#### Onde usar:

* **Script Manager** (`Window → Script Manager`)
* Pode salvar scripts custom (log, rename, export)
* Ghidra já vem com scripts prontos: rename by demangling, string extractor, syscalls mapper

***

### Plugins Úteis

* **ghidra-dark** → tema dark decente
* **ghidra-emotionengine** → suporte PS2 executáveis
* **ghidra-patcher** → patch binário avançado
* **BinExport** (do BinDiff) → diffing entre binários
* **Ghidrathon** → roda Python 3 nativo dentro do Ghidra

***

### Fluxo típico de análise (CTF/Malware)

1. **Imports/Strings** → acha APIs suspeitas (`VirtualAlloc`, `CreateProcess`, `socket`).
2. **Entrypoint** → segue pro `main` ou função de init.
3. **Graph** → mapeia fluxo das funções suspeitas.
4. **Renomear tudo** (funções, variáveis, labels).
5. **Stack/Heap Analysis** → procura buffers e cópias sem bounds-check.
6. **Decompiler** → ajusta tipos para reconstruir a lógica.
7. **XREFs** → entenda como os dados fluem.
8. **Scriptar** se análise repetitiva (ex: dump de offsets, renomear automaticamente).

***

### Instalação

* **Download**: [Ghidra NSA](https://ghidra-sre.org/)
* **Requisitos**: Java 17+
* **Plugins**: instale via `Extensions` no menu ou jogue na pasta `Ghidra/Extensions`

***

### Dicas Ninja

* **Highlight**: selecione um registrador ou variável → todos usos ficam destacados.
* **Rename Everywhere**: `L` ou `N` propaga nome no listing e decompiler.
* **Undo**: `Ctrl+Z` funciona para mudanças de análise.
* **Function ID Database**: Ghidra reconhece libs comuns automaticamente.
* **Export**: `File → Export Program` (ELF/PE com patches aplicados).

***

👉 Esse cheatsheet cobre **atalhos**, **análise de malware**, **exploit dev**, **scripting** e **plugins**. É praticamente um “curso de Ghidra em uma folha”.
