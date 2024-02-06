### Objetivo

Explorar uma geração inadequada de chaves para decifrar um criptograma, sem acesso à chave simétrica usada na sua criação.

### Contexto

o desafio CTF desta semana envolve efetuar um ataque brute-force para tentar adivinhar a chave a ser usada na encriptação e desencriptação dos blocos de textos ou cifrados. 

### Desafio

É fornecido um servidor que enviará uma flag com o aspeto habitual (flag{xxxxxx}) cifrada utilizando a operação counter mode. Para aceder a este servidor utilizamos o seguinte comando no terminal com a vpn ligada:

```c
$ nc ctf-fsi.fe.up.pt 6003
```

Este comando deu origem ao seguinte:

![img](https://cdn.discordapp.com/attachments/972271313802637392/1181037773524828230/image.png?ex=657f9a23&is=656d2523&hm=dd90c1bfa9fd0b1d9932246acbc711e70960bb059dac86f5fe6d773269819dd0&)

O servidor já disponibiliza o nonce e o ciphertext a ser usado e neste ciphertext está contida o valor da flag, no entanto está encriptado, logo para resolver este CTF necessitamos de desencriptar este texto.

#### Como conseguir usar esta ciphersuite para cifrar e decifrar dados?

Para desencriptar o ciphertext, nós primeiro fomos buscar o ficheiro disponibilizado pela plataforma de ctf:

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

KEYLEN = 16

def gen(): 
	offset = 3 # Hotfix to make Crypto blazing fast!!
	key = bytearray(b'\x00'*(KEYLEN-offset)) 
	key.extend(os.urandom(offset))
	return bytes(key)

def enc(k, m, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	encryptor = cipher.encryptor()
	cph = b""
	cph += encryptor.update(m)
	cph += encryptor.finalize()
	return cph

def dec(k, c, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	decryptor = cipher.decryptor()
	msg = b""
	msg += decryptor.update(c)
	msg += decryptor.finalize()
	return msg
```

Ao analisarmos este ficheiro apercebemos que se encontra dividido em 3 funções:

- gen(): responsável pela geração da key, retornando esta em formado de bytes (b'\x').
- enc(k, m, nonce): responsável pela encriptação da mensagem de texto normal. Leva como argumento uma key em bytes (k), uma mensagem de texto em bytes (m) e um nonce em bytes (nonce) e retorna a mensagem encriptada em bytes;
- dec(k, c, nonce): responsável pela desencriptação da mensagem encriptada. Leva como argumeno uma key em bytes (k), uma mensagem encriptada em bytes (c) e um nonce em bytes (nonce) e retorna a mensagem desencriptada em bytes;

Vistas estas funções para poder correr o código necessitamos ainda de instalar o ciphersuite para as poder usar. Para fazer isso fizemos o seguinte:

```c
$ pip3 install cipher suite
```

No entanto, mesmo após fazer isto aparece erros a compilar relacionado com a falta de um argumento opcional na função 'Cipher', que é usada quer na encriptação ou desencriptação da mensagem. Para isso importamos o 'default_backend' da biblioteca cryptography.hazmat.backends e mudamos o seguinte nas funções enc e dec também:

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os
from cryptography.hazmat.backends import default_backend

KEYLEN = 16

def gen(): 
	offset = 3 # Hotfix to make Crypto blazing fast!!
	key = bytearray(b'\x00'*(KEYLEN-offset)) 
	key.extend(os.urandom(offset))
	return bytes(key)

def enc(k, m, nonce):
	backend = default_backend()
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce), backend=backend)
	encryptor = cipher.encryptor()
	cph = b""
	cph += encryptor.update(m)
	cph += encryptor.finalize()
	return cph

def dec(k, c, nonce):
	backend = default_backend()
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce), backend=backend)
	decryptor = cipher.decryptor()
	msg = b""
	msg += decryptor.update(c)
	msg += decryptor.finalize()
	return msg
```

Após estas mudanças precisamos de saber o que é o counter-mode:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1181170845402533928/image.png?ex=65801612&is=656da112&hm=03d9968c6d53bb7847a2b49e1fe0dfecc1e64551ae3deb895fb5fe4be0dc45ee&)

Como vemos na imagem anterior, o nonce é um valor fixo que é normalmente disponibilizado para os utilizadores como é o caso deste ctf. É adicionado ao nonce um valor que incrementa à medida que muda de bloco de cifra. No entanto, é muito comum o valor inicial deste counter ser 0. O valor do nonce + counter é utilizado no bloco de cifra juntamente com a chave e o resultado é utilizado numa operação de XOR com o bloco de texto normal para originar o bloco de cifra final. Para a desencriptação o processo é o mesmo, mas só que invertido, pois o bloco de texto normal passa a ser o bloco de texto encriptado,...

Visto isto, reparamos que de facto para originar uma mensagem encriptada necessitamos de um nonce, uma chave e um bloco de mensagem normal, originando assim um bloco de texto encriptado e para desencriptar também necessitamos dos mesmos elementos. No entanto, nós a partir no servidor só conhecemos o nonce e o ciphertext, logo para desencriptar esta mensagem e encontrar a flag contida nela, necessitamos de descobrir a chave a ser usada e chamar a função dec.


#### Como conseguir fazer uso da vulnerabilidade que observei para quebrar o código?

Na função gen(), reparamos que existe uma vulnerabilidade associada à geração da chave de encriptação do bloco de cifra:

```python
def gen(): 
	offset = 3 # Hotfix to make Crypto blazing fast!!
	key = bytearray(b'\x00'*(KEYLEN-offset)) 
	key.extend(os.urandom(offset))
	return bytes(key)
```

A chave tem 16 bytes de tamanho que pode ser visto através da variável global seguinte:

```python
KEYLEN = 16
```

No entanto, ao gerar esta chave na função gen(), é enchido os primeiros 13 bytes da variável key com b'\x00' e só os últimos 3 bytes são aleatórios:

```python
key = bytearray(b'\x00'*(KEYLEN-offset)) 
key.extend(os.urandom(offset))
```

Para adivinhar a chave só necessitamos então de lançar um ataque de brute-force a tentar todas a permutações de 3 bytes para os últimos 3 bytes da chave:

```python
nonce = binascii.unhexlify("e5043a2f3354f26dc492f03def06fc53")    
    
ciphertext = binascii.unhexlify("d53430c7756898a5f909befc4528c33e96b32cd78d4d877f93f2d410ee4a017048f79c341630fd")


for i in range(256):
    for j in range(256):
        for k in range(256):
            key1 = bytes([i, j, k])
            key = bytearray(b'\x00'*13)
            key.extend(key1)
            plaintext = binascii.hexlify(dec(key, ciphertext, nonce))
            if(plaintext.startswith(b'666c6167')):
		print("The key is: ", binascii.hexlify(key), '\n')
                print(plaintext, '\n')
```

Primeiro guardamos o nonce e o ciphertext em variáveis e usamos a função hexlify que leva um string de bytes como parâmetro e retorna os mesmos bytes mas em formato b'\x'. Para usar esta função tivemos que dar import do seguinte:

```python
import binascii
```
Após estas variáveis, nós usamos três loops que são para gerar todos os valores possíveis para 3 bytes. 1 byte em cada loop sendo que um byte tem 8 bits então são 256 possíveis permutações para cada byte, logo têm que haver 3 loops de 256 cada.

Dentro de cada um destes loops usamos o seguinte procedimento para gerar uma key a brute-force:

```python
key1 = bytes([i, j, k])
key = bytearray(b'\x00'*13)
key.extend(key1)
```

Essencialmente vai gerar 13 bytes com valores b'\x00' e vai acrescentar a esses bytes mais 3 bytes aleatórios.

Agora que obtemos uma chave possível, podemos chamar a função dec com os nossos dados:

```python
plaintext = binascii.hexlify(dec(key, ciphertext, nonce))
```

Nota: usamos o hexlife, pois queremos converter o texto decifrado de formato b'\x' para um string de bytes.

#### Como conseguir automatizar este processo, para que o ataque saiba que encontrou a flag?

A automatização já é feita através dos loops, que vão gerando todas as permutações possíveis de chaves a ser usadas e depois usar essa chave na função dec para receber o plaintext.

Agora nesta fase, basta-nos imprimir todos os plaintexts e ver se um deles contém a flag. No entanto, nós usamos uma forma de encontrar o plaintext que é beneficial para nós (o plaintext da flag). Para isto fomos buscar os valores de flag em hexadecimal:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1181042293659152424/image.png?ex=657f9e59&is=656d2959&hm=475dd634c7260a50f88f0af3830e553d259046f236fef2b061a98307ea077643&)

E usamos o seguinte código para localizar a flag duma maneira mais eficaz:

```python
if(plaintext.startswith(b'666c6167')):
	print("The key is: ", binascii.hexlify(key), '\n')
        print(plaintext, '\n')
```

Este código vai ver se existe no plaintext algum text que inclui a palavra flag no início e se incluir então vai imprimí-la. Após correr o código origina o seguinte resultado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1181928554338975824/image.png?ex=6582d7be&is=657062be&hm=983cfaae79a931f68823944a65f68d0562e5478b7d4131aa16dcbd7d9ae9b578&)

Após converter este valor hexadecimal para texto obtemos:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1181042133151531038/image.png?ex=657f9e33&is=656d2933&hm=43c8ece3d2fd06803273c58e4829bde14b5768d9da15d10e8dd722a93e8a0621&)











