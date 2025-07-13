# CubeMadness1ament

**Categoria:** Game Hacking / Memory Manipulation\
**Dificuldade:** Fácil\
**Ferramenta usada:** Cheat Engine\
**Plataforma:** Windows\
**Objetivo:** Coletar 20 cubos para completar o desafio. O jogo, no entanto, exibe apenas 6 cubos visíveis.

***

<figure><img src="../../../.gitbook/assets/Pasted image 20250713120222.png" alt=""><figcaption></figcaption></figure>

### 🧠 1. **Análise Inicial com DiE (Detect It Easy)**

A primeira etapa foi verificar o executável usando o [DiE (Detect It Easy)](https://ntinfo.biz/) — uma ferramenta para análise estática que detecta o packer, compilador, e outros metadados do binário.

> **Resultado:**\
> O binário não estava ofuscado ou compactado com packers conhecidos (como UPX ou ASPack). Nenhuma informação relevante foi extraída dessa análise estática.

<figure><img src="../../../.gitbook/assets/Pasted image 20250713121828.png" alt=""><figcaption></figcaption></figure>

### 🕹️ 2. **Execução e Observações no Jogo**

Ao executar o game:

* O objetivo indicado era **coletar 20 cubos**.
* Contudo, **apenas 6 cubos** estavam visíveis na interface.
* Isso levanta a suspeita de que os cubos restantes são virtuais ou precisam ser **manipulados diretamente na memória**.

<figure><img src="../../../.gitbook/assets/Pasted image 20250713121938.png" alt=""><figcaption></figcaption></figure>

### 🧪 3. **Manipulação com Cheat Engine**

#### 🔍 **3.1. Localizando a Variável de Contagem de Cubos**

Iniciamos a investigação dinâmica com o **Cheat Engine**:

1. Abrimos o jogo e o Cheat Engine simultaneamente.
2. Selecionamos o processo do jogo dentro do Cheat Engine.
3. Coletamos 1 cubo no jogo e procuramos o valor `1` na memória.
4. Repetimos a coleta e a busca (Next Scan → valor `2`, depois `3`, e assim por diante).
5. Após alguns ciclos, **isolamos o endereço** da variável de contagem de cubos.

***

#### 🛠️ **3.2. Modificando o Valor**

Com a variável localizada:

* Simplesmente **alteramos seu valor de `6` para `20`**.
* Instantaneamente o jogo **registrou a vitória**, como se todos os cubos tivessem sido coletados.

<figure><img src="../../../.gitbook/assets/Pasted image 20250713120746.png" alt=""><figcaption></figcaption></figure>

***

### 🎯 4. **Resultado: FLAG!**

O jogo não validava logicamente a presença dos cubos, apenas verificava o valor da variável de contagem. Isso nos permitiu **burlar a mecânica do jogo com uma simples edição de memória e conseguir a flag**.

***

### ✅ 5. **Conclusão e Boas Práticas**

Este desafio exemplifica como:

* Muitos jogos **não protegem a integridade de suas variáveis em tempo de execução**.
* Ferramentas como Cheat Engine são **poderosas para prototipagem, debugging e análise de engenharia reversa**, mesmo sem recorrer à desmontagem completa.

> 💡 **Dica Profissional:**\
> Em jogos com _anti-cheat_, essa abordagem pode não funcionar devido à proteção contra leitura/escrita na memória. Análises mais profundas exigiriam o uso de **debuggers como x64dbg ou Ghidra** para compreender a lógica interna.
