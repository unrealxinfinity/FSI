## Secret-Key Encryption

### Overview

Neste lab, vamos ver vulnerabilidades e explorações associadas a encriptação e desencriptação de mensagens. Existem mais do que uma forma de encriptar e desencriptar mensagens, mas neste lab vamos ver este processo através de um uso de uma chave pública.

Uma chave pública é usada para as duas ações: encriptação e desencriptação. O que torna este processo muito vulnerável e em risco de vários ataques.

### Lab Environment

Na página de Secret-Key Encription no seed labs, encontra-se um zip que ao ser descomprimido terá este aspeto:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179173313201836082/image.png?ex=6578d1b9&is=65665cb9&hm=cd94063c8a8a2a5717cccee70905e45070f589529c6cbbcdee52cd13cd6f3699&)

nota: nestas tasks não vamos utilizar containers, por isso o uso do docker não é necessário.

### Task 1: Frequency Analysis

Nesta task vamos tentar desencriptar um texto inglês que se encontra no ficheiro cyphertext.txt na diretoria files:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179195727109427200/image.png?ex=6578e699&is=65667199&hm=bb07a008077b917af7c6669ac0225d418ea3eb60396887fc2c187f8e544c8638&)


O conteúdo do ficheiro é o seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179195886862086357/image.png?ex=6578e6bf&is=656671bf&hm=6179be2633fe29b0fb025704ecb565dc8a16ab8058f1eadaedf00df5f2e775b1&)

Tal como verificamos, este texto encontra-se encriptado. Para o desencriptar corremos o código de python que também se encontra na mesma diretoria e que tem como nome freq.py. Este código vai contar o número de ocurrências de letras ou conjuntos de letras do ficheiro cyphertext.txt e mostrá-lo no terminal:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179196444440281239/image.png?ex=6578e744&is=65667244&hm=0a69c3e3d396a1bb48a49f330635fb971785aa4fb7a39d9ad76263fd3222d3be&)


Após analisar estes dados, como a encriptação e a desencriptação é formada através da mesma chave, ou seja, para encriptar uma letra ou um conjunto de letras será bidirecional. Logo, podemos pesquisar os conjuntos de letras do alfabeto inglês mais utilizadas em média em textos e sustituir os conjuntos de palavras encriptados do texto de ciphertext.txt com essas palavras. Primeiro substituímos o conjunto de letras com 3 letras seguidas, depois com 2 e depois com 1 para adicionar todas as letras do alfabeto.

Para descriptar o texto utilizamos o seguinte comando:

```c
$ tr 'nyvxuqmhtipaczlgbredsfkojw' 'etaonsirhldcmuwbfgpykvxjqz' < ciphertext.txt > out.txt
```

E no ficheiro out.txt observamos o texto ciphertext.txt desencriptado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179198232417550417/image.png?ex=6578e8ef&is=656673ef&hm=d78ffb6352a81081449ef509f5f2ff2591224ce244856f9818ee210ba8e53d1e&)

### Task 2: Encryption using Different Ciphers and Modes

Usando o comando 'openssl enc' conseguimos encriptar ou desincriptar ficheiros. Este comando pode ser usado desta forma:

```c
$ openssl enc -ciphertype -action -in plain.txt -out cipher.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

Sendo que os valores após 'openssl enc' têm o seguinte significado:

- ciphertype: o modo de encriptação e desencriptação a ser usado;
- action: 'e' se for para encriptar e 'd' se for para desencriptar; 
- in: o ficheiro a seguir é a origem da ação;
- out: o ficheiro a seguir é o destino da ação;
- K: o número a seguir é a chave a ser usada na ação;
- iv: o número a seguir é a valor no vetor de incialização.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180873288042418278/image.png?ex=657f00f3&is=656c8bf3&hm=31209c4c73afbfc0c8621fb4731e7ed64823247ebef98bf10c53d4ac9defc506&)

No processo de encriptação e desincriptação, são utilizados blocos de cifra (Cipher Blocks) que são responsáveis para encriptar/desencriptar um bloco de texto. No ato de tentar encriptar um texto, será preciso usar muitos blocos de cifra. Estes blocos de cifra, no modo usado anteriormente só utilizavam uma chave para encriptar ou desencriptar um bloco de texto, que se tornava vulnerável a ataques de análise de frequência.

Vamos ver que outros 3 modos podemos utilizar para esta task:

#### Cipher Block Chaining (CBC)

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180871747822694440/image.png?ex=657eff84&is=656c8a84&hm=18da84c8bf151804b3ba0c1af53c60692b1ff8ce8663c0fc552afbb568272216&)

Neste modo, reparamos que cada bloco de texto normal (Plaintext), é usado numa operação de XOR com o bloco de cifra anterior, sendo que o primeiro bloco de texto normal, como não tem bloco de cifra anterior, então é usado um vector de inicialização dado como input no comando 'openssl enc' após o -iv. Desta forma dois blocos de texto que são iguais em texto normal não terão o mesmo valor após a encriptação, pois o bloco de cifra anterior de cada um deles são diferentes.

Utlizando os seguintes comandos 'openssl enc' podemos testar da seguinte forma:

```c
#encriptação
$ openssl enc -aes-128-cbc -e -in out.txt -out in.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180875263882571907/image.png?ex=657f02ca&is=656c8dca&hm=b1daf7bce4efbe557edbccea1d8c2a0aee6040fc1ddcf9ba3843f314e0e985da&)

```c
#desencriptação
$ openssl enc -aes-128-cbc -d -in in.txt -out exit.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180876637429051432/image.png?ex=657f0411&is=656c8f11&hm=639475432aa88c36a517d9cee54346b43532a20f21d2c3080597aab3cd1953c0&)

Nota: Nós estamos a usar o ficheiro out.txt, que é o resultado da task 1, como ficheiro de texto original, dois ficheiro criados por nós, in.txt, como ficheiro que terá o out.txt encriptado e exit.txt que terá o resultado de in.txt desencriptado.

#### Cipher Feedback (CFB)

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180876818610409542/image.png?ex=657f043d&is=656c8f3d&hm=1a74f2ab7a25ec571e4885f3e2362042b37e4f4f9a4171440429b1d0bf82a9da&)

Neste modo, reparamos que em vez de ser o bloco de texto normal a ser usado no bloco de cifra, está a ser usado o texto cifrado do bloco de cifra anterior (depois de levar XOR com o bloco de texto normal) e depois o resultado está a ser usado numa operação de XOR com o bloco de texto normal para originar a desencriptação final. Novamente é utilizado um vector de inicialização, pois no primeiro bloco de cifra não existe um bloco anterior por isso é dado como input o valor do bloco anterior.

Utlizando os seguintes comandos 'openssl enc' podemos testar da seguinte forma:

```c
#encriptação
$ openssl enc -aes-128-cfb -e -in out.txt -out in.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180877935952023562/image.png?ex=657f0547&is=656c9047&hm=e47d8f62b56591380e45680702381882d8a180cf7bbe89658a4fd28424e72e84&)

```c
#desencriptação
$ openssl enc -aes-128-cfb -d -in in.txt -out exit.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180876637429051432/image.png?ex=657f0411&is=656c8f11&hm=639475432aa88c36a517d9cee54346b43532a20f21d2c3080597aab3cd1953c0&)

#### Output Feedback (OFB)

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180878464308482068/image.png?ex=657f05c5&is=656c90c5&hm=b95896dce99f5a741da8165a261522c68c8f14be3ac341ca44af38d6f48b2017&)

Neste modo, reparamos que é essencialmente o mesmo procedimento que o modo anterior (CFB), no entanto neste modo o texto a ser usado no bloco de cifra é o resultado do texto cifrado do bloco anterior antes de levar XOR com o bloco de texto normal anterior (no CFB é depois de levar XOR).

Utlizando os seguintes comandos 'openssl enc' podemos testar da seguinte forma:

```c
#encriptação
$ openssl enc -aes-128-ofb -e -in out.txt -out in.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180879572678492190/image.png?ex=657f06cd&is=656c91cd&hm=256c5661e4a3b5a9fb05e3882a253d0d7297bf680c9015847c8a206fb920e60e&)

```c
#desencriptação
$ openssl enc -aes-128-ofb -d -in in.txt -out exit.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180876637429051432/image.png?ex=657f0411&is=656c8f11&hm=639475432aa88c36a517d9cee54346b43532a20f21d2c3080597aab3cd1953c0&)

### Task 3: Encryption Mode - ECB vs CBC

Nesta task vamos tentar encriptar uma imagem que se encontra na diretoria Files e que se chama pic_original.bmp:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180909153909223554/image.png?ex=657f225a&is=656cad5a&hm=a94604d65e1ba17ee4afc95fa82363abe2dfaf8715f82694206fb3fbafa34e95&)

Vamos tentar encriptar usando o modo ECB que foi o que usamos na task 1 e vamos compará-lo com o modo CBC:

Conseguimos encriptar a imagem no modo ecb através do seguinte comando:

```c
$ openssl enc -aes-128-ecb -e -in pic_original.bmp -out cipherpic.bmp -K 00112233445566778889aabbccddeeff
```

Nota: No modo ECB não é usado um vector de inicialização, pois o processo de encriptação e desencriptação deste modo não envolve blocos de cifra anteriores.

Para poder visualizar a imagem temos que usar os seguintes comandos:

```c
$ head -c 54 pic_original.bmp > header
$ tail -c +55 cipherpic.bmp > body
$ cat header body > new.bmp
```

Agora na diretoria files parece o new.bmp que é o ficheiro encriptado de pic_original.bmp:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180248539981418758/image.png?ex=657cbb1b&is=656a461b&hm=ccf8aaadd0bbf3f80e69c20ee7c0e9a93d2bc1281a909b5ea52d312adabb9da6&)

E se abrirmos o new.bmp conseguimos ver o seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180248637167632444/image.png?ex=657cbb33&is=656a4633&hm=1b9e0fb7687d00c16ee8f42afa592cdb57b6fce8f0f359c5a87dfbd867aa58c0&)

Ao compararmos esta imagem com a imagem original de pic_original.bmp, reparamos que as cores mudaram, mas a estrutura da imagem parece semelhante. Isto acontece, pois no modo ECB, se dois blocos forem iguais, as suas encriptações também originarão blocos iguais, logo apesar das cores mudarem, mudam todas para a mesma cor encriptada.


Se experimentarmos o mesmo, mas agora em modo CBC:

```c
$ openssl enc -aes-128-cbc -e -in pic_original.bmp -out cipherpic.bmp -K 00112233445566778889aabbccddeeff
```

```c
$ head -c 54 pic_original.bmp > header
$ tail -c +55 cipherpic.bmp > body
$ cat header body > new.bmp
```

Reparamos que a imagem encriptada é a seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180249941864611851/image.png?ex=657cbc6a&is=656a476a&hm=506ca1f2f58de241a33dc96a259001932059360c91740a0f1327eaf87c7dc501&)


Neste modo, se dois blocos são iguais, então as suas encriptações irão originar blocos differentes, logo apesar da cor do bloco original ser a mesma, irá originar uma cor differente cada vez que é encriptada.
