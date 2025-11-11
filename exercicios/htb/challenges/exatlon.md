# Exatlon

Chall de reverse:

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

o DiE mostra a versão 3.95 do UPX como packer, além de:

1. **Overlay detectado**:
   * `Overlay: Binary[Offset=0x00093000, Tamanho=0x0001a394]`\
     Overlays são comuns em binários empacotados, onde metadados/código adicional são anexados ao final do executável.

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Alta entr0pia:

<figure><img src="../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Testando com UPX:

<figure><img src="../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

desempacotando:

<figure><img src="../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

checksec:

<figure><img src="../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

