# rabin2

### Visão geral

`rabin2` faz **inteligência estática** relâmpago de binários (ELF/PE/Mach‑O): metadados, seções, símbolos, imports/exports, strings.

### Metadados e arquitetura

```bash
rabin2 -I hello      # infos gerais (arch, bits, libs, entry)
rabin2 -A hello      # resumo completo (agregado)
```

### Strings, símbolos, imports/exports

```bash
rabin2 -z hello      # strings (com offsets)
rabin2 -s hello      # símbolos (funções/variáveis)
rabin2 -i hello      # imports
rabin2 -x hello      # exports
```

### Seções, relocs, entrypoint

```bash
rabin2 -S hello      # seções (nome, vaddr, size)
rabin2 -r hello      # relocações
rabin2 -e hello      # entrypoint
```

### Casos úteis

```bash
# Triage rápido
a) file target
b) rabin2 -I target
c) rabin2 -z target | head
c) rabin2 -s target | head
```

**Dicas**

* Combine com `checksec` para NX/PIE/Canary.
* Procure seções suspeitas (`.upx`, `.adata`, etc.).
