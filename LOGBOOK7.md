## Format String


### Overview

Neste seed labs iremos identificar e explorar vulnerabilidades associadas à utilização do printf() com argumentos passados através de um input malicioso do utilizador.  

A função printf utiliza uma estrutura de dados va_list de forma a percorrer o número de argumentos passados, que podem ser um número variado de argumentos passados por esta função.      

Ao colocar no primeiro argumento, que é o string a ser formatado, um "%", estaremos a indicar à va_list para aceder a um próximo valor da stack de forma a encontrar o registo pretendido para ser printado.   

Existem vários tipos de formatos em que a va_list é movimentada. Se o user colocar um %d, então va_list irá movimentar-se 4 bytes para cima da stack de forma a aceder ao próximo argumento.


### Environment Setup 

#### Turning of Countermeasure

Em primeiro lugar desligamos a randomização do espaço de endereços, pois com este modo de proteção ligado tornaria-se mais difícil adivinhar a localização de endereços na stack.

Esta etapa foi concluida com o seguinte comando: 
```c
$ sudo sysctl -w kernel.randomize_va_space=0
```

#### The Vulnerable Program

Após descomprimir o zip disponibilizado pelo seed labs, deparamo-nos com três diretorias:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171462267154477169/image.png?ex=655cc441&is=654a4f41&hm=2aeb60336c77c2c5513416bd4d08e9ae41960eecdcdb664e138a2157ad568c9e&)

A pasta attack-code contém ficheiros de python (.py), que servem como auxílio para executar código malicioso.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171530335331287120/image.png?ex=655d03a5&is=654a8ea5&hm=070dfffe3ecdf12b5e05b58c28c603fdabe40b4449aa204a9ac4b500914fbfac&)


A pasta server-code contém o format.c que é o programa vulnerável que o servidor irá disponibilizar para os seus utilizadores. Após correr make irá ter este aspeto:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171712478170841169/image.png?ex=655dad48&is=654b3848&hm=76619228e4cb0b2557ff0fd0f50ea726566878193caa61e45c23996eef7524b1&)


A pasta fmt-containers contém os containers do docker e após fazer make install na pasta server-code, passamos o programa de format para esta pasta, essencialmente permitindo o docker de o usar para o servidor.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171713064492609567/image.png?ex=655dadd3&is=654b38d3&hm=fa5f9d66beda1b8e45414c4dd992dbf600c1a64966aebe6c74e46f350892bacf&)

#### Container Setup and Commands

Após ter feito make e make install, fomos para a diretoria com o ficheiro yaml do docker e corremos os comandos de docker para inicializar o servidor:

``` c
$ dcbuild
$ dcup
```

Após isto, abrimos um segundo terminal que servirá de utilizador do servidor, em que agora podemos comunicar com o servidor correndo o programa format. No entanto, primeiro temos que descobrir o estado do docker para saber onde é que ele está a correr o servidor. Conseguimos descobrir o estado do docker através deste comando:

``` c
$ dockps
```
Este comando revelou os diferentes hosts do servidor e o onde é que este está a ser corrido

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171714983156002886/image.png?ex=655daf9d&is=654b3a9d&hm=aa1f4f10c272fe53e82be3c21541f0f6257c6690ac820812ad0d67393fb8d902&)

Observamos que o server do hostA está a correr em 10.9.0.5, que é o que vamos usar, pois aí corre o format.c em formato 32 bits.

### Task 1: Crashing the Program

Para connectar ao servidor, no terminal de utilizador, recorremos ao comando de netcat juntamente com o local onde corre o servidor e a sua porta (neste caso a porta 9090). Para interagir com o servidor utilizamos um comando que printa no terminal antes de chamar o nc:

``` c
$ echo hello | nc 10.9.0.5 9090
```

Desta forma o servidor recebe o nosso input que é "hello", e irá correr o programa format.c com esse input, que posteriormente será esse mesmo input que será usado dentro da função printf().

Depois de usar Control C para parar a comunicação com o servidor, observamos no terminal do servidor que tudo correu bem

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171719420461199391/image.png?ex=655db3bf&is=654b3ebf&hm=f5805d6edc21cca225bb796e4ff8c2f5b936e58b7262d96076d5f4313b42cd3a&)

Esta mensagem indica-nos que o programa correu sem qualquer erro ou segmentation fault, ou seja, o servidor não foi mandado abaixo.


No entanto, nesta task foi-nos pedido para mandar o servidor abaixo através do input de um utilizador. Para conseguir fazer isto é necessário saber como é que um input na função printf() cause um programa para ir abaixo.

A função printf guarda os argumentos passados na stack de forma inversa, ou seja, de baixo para cima. Efetivamente se escrever estas duas linhas de código:

``` c
int id = 100, age = 25; char * name = "Bob Smith";
printf("ID: %d, Name: %s, Age: %d\n", id, name, age);
```

Então a stack terá este formato:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171732012525158410/image.png?ex=655dbf79&is=654b4a79&hm=68712e01d0c62a5fa55d2852fbcc5a085b3b60d98df01a26ab2fcc1d023430ad&)

A estrutura de dados chamada va_list é responsável por cada vez que encontrar um delimitador "%" dentro do format string, então irá avançar com um offset correspondente dependente ao tipo de variável encontrada. Se for um %d então avança 4 bytes para cima, se for um %lf então avança 8 bytes,...

No entanto, a va_list não sabe quando é que o número de argumentos acaba, ou seja, sempre que ele encontra um delimitador "%", ele vai avançar pela stack.

Para mandar abaixo o servidor simplesmente mandamos como input vários "%s", pois os char * não são guardados na stack, são guardados em memória, e na stack encontram-se os endereços na memória a apontar para esse char *, logo o %s vai buscar um endereço de memória associado a esse char *. Visto isso, naturalmente, se o endereço que tentamos aceder for inválido então o programa vai abaixo.

 Input dado:
``` c
echo %s%s%s%s | nc 10.9.0.5 9090
```

Resposta do servidor:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171734082196095016/image.png?ex=655dc166&is=654b4c66&hm=fc77738d24ead013fccc094ad82f81fbe74cc45737f18d7ae0500dde0e9e764f&)

O servidor não retornou "Returned Properly", logo foi-nos claro que foi abaixo, pois o servidor foi formatado de forma a se for abaixo então simplesmente não retornava como deve ser em vez de literalmente ir abaixo.

### Task 2: Printing Out the Server Program's Memory

#### Task 2.A:Stack Data

Nesta task, foi-nos pedido para printar no lado do servidor um input nosso, com o objetivo de conseguir ler da stack a partir do servidor com input de utilizador do servidor.

Para isto usamos o build_string.py localizado na pasta attack-code. 

Formámos uma payload e escrevemo-la num ficheiro chamado badfile, colocando nesta payload um input nosso genérico, no nosso caso 1234 nas primeiras 4 posições, e nas seguintes colocámos 64 ".%.8x". O primeiro ponto serve para separar cada valor, para ser mais fácil a leitura, o "%.8x", indica para printar 8 carateres de cada vez. Este processo é essencialmente uma tentative e erro até encontrar o valor necessário de "%x" até obtermos o nosso input a partir da stack.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171737049620037673/image.png?ex=655dc42a&is=654b4f2a&hm=55210bccdc77d9dba25f1092797921f1a57e96c7e818163c542bbcd4bca51fd5&)

Compilámos o código de build_string.py gerando assim o badfile e agora passamos este badfile como input para o servidor:

``` c
$ cat badfile | nc 10.9.0.5 9090
```

obtemos assim esta resposta do servidor:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171737420203556884/image.png?ex=655dc482&is=654b4f82&hm=1a3dda6ccc72929ca56fe927e3ca92eae953447a70fb38297b370b88a6f1ee39&)

Sendo que o último valor printado corresponde ao nosso input mas em hexadecimal (0x34333231), e nota que está em little endian, por isso é que tem aspeto invertido.

#### Task 2.B: Heap Data

Nesta task, pedem-nos para printar uma mensagem secreta da heap. A mensagem é um char * buffer, e este é guardado na heap, no entanto o seu endereço é guardado na stack e para ler o valor a que o endereço do char * está a apontar temos que usar um "%s". Logo usando a mesma lógica do passo anterior, apenas temos de concatenar um "%s" no final do payload, e claro providenciar o input no printf(), que é o endereço da mensagem secreta, que é providenciada pelo servidor:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171742577800794112/image.png?ex=655dc950&is=654b5450&hm=de0a62b99109dc9376ab237c0d4578dce500112a29132dcebfcf5595281a3f05&)

O nosso build_string.py tem este aspeto:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171742756524265533/image.png?ex=655dc97b&is=654b547b&hm=0432af686c36ae528df14cf5495a363f69a6d30506b27595a261ad0bb484be41&)

E escrevemos assim no terminal do utilizador do servidor:

``` c
$ cat badfile | nc 10.9.0.5 9090
```

Obtivémos assim a resposta do servidor:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171743130823950346/image.png?ex=655dc9d4&is=654b54d4&hm=39c8d83a9feb338d274c8782b700d5425a0fff0525bdc9ad4cdd99c616632a79&)


### Task 3: Modifying the Server Program's Memory

Nesta task, temos que mudar o valor de uma variável cujo endereço é providenciado pelo servidor:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171744325865058344/image.png?ex=655dcaf1&is=654b55f1&hm=9f3afb696c8fd5315ce73ea6b6bb753369e7ea5d8a1c6043860885bd9c235c2d&)

#### Task 3.A: Change the value to a different value

Nesta subtask temos de mudar o valor the uma variável. Para fazer isto vamos simplesmente usar a mesma abordagem da Task 2 para printar a data da heap. No entanto, nesta subtask temos que mudar o valor e não lê-lo, logo vamos simplesmente substituír o nosso "%s" por "%n", visto que "s" é usado para ler de um endereço e "%n" é usado para escrever no endereço o número de chars que foram escritos no format string desde o início até o "%n" correspondente.

O nosso build_string.py atualizado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171745075051646986/image.png?ex=655dcba3&is=654b56a3&hm=27a0b7d40d33f8885e8dadb5ed4a2d35468e455f3bf5f7a4225862a0e8088d93&)

Usámos o mesmo comando das tasks anteriores:

``` c
$ cat badfile | nc 10.9.0.5 9090
```

A resposta do servidor:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171745367881154630/image.png?ex=655dcbe9&is=654b56e9&hm=88d019230a72b0f3aa73726c8a63d334c11b293f75011f552dbf316b6589e9c0&)

Nota: Repara como a o valor da target variable (before) é diferente do valor da target variable (after), indicando que a nosaa abordagem funcionou.

#### Task 3.B: Change the value to 0x5000

Nesta subtask, temos que não só mudar o valor da target variable, mas como também temos de a mudar para um valor igual a 0x5000. Como o "%n" escreve o número de chars que foi escrito no format string até o "%n" correspondente, então temos que colocar 20480 chars antes do "%n", sendo que 0x5000 é igual a 20480 em decimal.

Anteriormente estávamos a usar ".%.8x" para printar 8 chars no formatted string, no entanto se queremos printar 20480 antes do "%n" então temos que adicionar mais, ou simplesmente somar o necessário até lá:

Número a somar = 20480 - 571         // 19909

571 é o número de carateres que foi printado na task 3.A, logo só precisamos de somar os restantes ao que já estava da task 3.A:

build_string.py atualizado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171754093245644851/image.png?ex=655dd409&is=654b5f09&hm=7547080d83739299db019a90940d2871f12b3b1f3a6361d9bdb84a148c7d98dc&)


Nota: o resultado a adicionar é 19909, no entanto acrescentamos tiramos um ".%.8x" para somar esse valor, logo tivemos que adicionar mais 8 ao 19909 tornando-se assim 19917.

Usamos o mesmo comando para introduzir o input no servidor:

``` c
$ cat badfile | nc 10.9.0.5 9090
```

Resposta do servidor:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1171754840859365428/image.png?ex=655dd4bc&is=654b5fbc&hm=0737ab1ea123722e86989209fe5fcedd9a4d03d4b3b3f59d0b812cb9ffccf388&)



