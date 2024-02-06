# CTF Semana 7

## Desafio 1 

> O primeiro desafio consiste em ler uma determinada flag acedendo a sua variável na memoria usando a vulnerabilidade format string.

### Passo 1: 

Verificar o ficheiro main.c e exploit_example.py e ficamos a saber a estrutura do código para saber mais ou menos como a stack poderia estar organizada e que o programa em python consegue parar o instruction counter para usarmos debugger "GDB".
Verificar os mecanismos de segurança garantidos pelo ficheiro executável "programa". Neste caso verificamos que ele não tem PIE, ou seja, posição do buffer que contẽm o ficheiro com a flag será igual para o servidor e máquina do cliente.

![img1](https://cdn.discordapp.com/attachments/1153998326274994216/1177022852420075561/image.png?ex=6570fef4&is=655e89f4&hm=ef0d019b5ea65c1f43e4303c47a7de700f4b0db168d3772830090b3c6a484569&)



### Passo 2:

Usamos "GDB" localmente com o pid dado pelo output do programa python enquanto pausado para buscar o endereço da variável do buffer que queremos aceder na memória que contém o contúdo da flag. 
No nosso caso vimos caso a caso seguindo o flow do gdb em vez de usar p &NomeDaVariavel o que poupava monte de trabalho.

![img2](https://media.discordapp.net/attachments/1153998326274994216/1176936742692061254/image.png?ex=6570aec1&is=655e39c1&hm=1f31583e8ae9909d70ec9a50684a014e7e96a980158b04851a59dd98f1f2d465&=&format=webp)

### Passo 3:

Substituir o endereço encontrado em EAX mostrado na foto anterior no programa de python, colocar a variàvel "LOCAL" a falso para aceder ao servidor e correr a script.

![img3](https://media.discordapp.net/attachments/1153998326274994216/1176936743325413416/image.png?ex=6570aec2&is=655e39c2&hm=0acdeef81a545e1fee75f5bf955289393daf14668edce72002c634cb41533ea8&=&format=webp)

### Resultado :
 
![img4](https://cdn.discordapp.com/attachments/1153998326274994216/1176931860090191902/image.png?ex=6570aa35&is=655e3535&hm=8df6838e5f6febfd85c5ec63ede8c94e34d8b4ad96af21ed3c9a08650477f80e&)


## Desafio 2 

> O segundo desafio consiste em escrever para um endereço específico usando fortmat string. Este desafio é parecido com o anterior mas muda só no input que damos ao programa.

### Passo 1: 

Ver o codigo fonte no ficheiro local em "main.c". Verificar os mecanismos de segurança implementados pelo programa a correr "program" e PIE estava desativo como anteriormente.

![img1](https://media.discordapp.net/attachments/1153998326274994216/1177270255664496730/image.png?ex=6571e55d&is=655f705d&hm=de54a5f094bcfcd56da85ec6c1d07be2014bcc70dddc590850e562594697421f&=&format=webp&width=1440&height=328)

### Passo 2:

Em main.c vimos que precisamos de alterar valor de uma variável "key" para poder entrar no if condition que contém código que invoca a shell no servidor. Com isso podiamos ver o conteúdo do ficheiro que tem a flag.

Usar "GBD" no processo do "program" e encontrar o endereço da variável que queremos alterar:
![img2](https://cdn.discordapp.com/attachments/1153998326274994216/1177274886704607373/image.png?ex=6571e9ad&is=655f74ad&hm=12594995427f7d4dba7376b10aa05cc51f5b452a3c4be1844b3417832f38cbcd&)
![img3](https://media.discordapp.net/attachments/1153998326274994216/1177274886994018416/image.png?ex=6571e9ad&is=655f74ad&hm=bfd8d765b2784381f18937e2a4cc6372a214bb5a75212cd35e53183bcbdc696e&=&format=webp)

### Passo 3:

Editar programa em python "exploit_example.py" e escrever o format string da seguinte maneira:
![img4](https://media.discordapp.net/attachments/1153998326274994216/1177270254838235237/image.png?ex=6571e55d&is=655f705d&hm=c1becbbb34c8f29e65043dc148f0f9ec9fe6498995f886c27f5f703f65e084f1&=&format=webp&width=1440&height=601)

### Resultados:

Obtivemos a seguinte flag depois de aceder a shell em ctf-fsi.fe.up.pt na porta 4005, ao mostrar conteudo de um ficheiro:
![img5](https://media.discordapp.net/attachments/1153998326274994216/1177270254838235237/image.png?ex=6571e55d&is=655f705d&hm=c1becbbb34c8f29e65043dc148f0f9ec9fe6498995f886c27f5f703f65e084f1&=&format=webp&width=1440&height=601)

## Desafio 3 

> Este desafio é semelhante ao passado, com diferenca de que o endereco que a chave da acesso a bash contém \x20 que é reconhecido conhecido como outra coisa em python

### Passo 1:

Resolver o problema com o endereço descendo um endereço abaixo da "key" em 0x0804b31f e escrever 0xbeef00 nesse endereço para que fique 0xbeef no endereço correto 0x0804b320

### Resultado:

![img6](https://cdn.discordapp.com/attachments/1153998326274994216/1177370743344996442/image.png?ex=657242f3&is=655fcdf3&hm=8220d619cd274b3d5bad5c2ef5102a944122e0632daef1f8ef89651d5081e4e7&)
