# RSA
> O objetivo principal deste ctf era obter a flag a partir da chave pública que é nos fornecido e através do algoritmo de RSA calcular a mensagem a partir do ciphertext

### Passo 1 - Perceber o algoritmo de RSA e como calcular valores

Ao analisar o codigo percebemos que o challenge.py fornece nos uma função essencial para descodificar o ciphertext, dec(ciphertext,d,n) em que os seus argumentos sao :
- **ciphertext** : mensagem a decifrar;
- **d**: número da chave privada que temos que obter a partir da chave publica (e,n);
- **n**: parte da chave pública (e,n);

```py
def dec(y, d, n):
    int_y = int.from_bytes(unhexlify(y), "little")
    x = pow(int_y,d,n)
    return x.to_bytes(256, 'little')
``` 


Com as informações fornecidas no moodle percebemos para calcular o tociente de euler , em outras palavras fi do **n** que faz parte da chave pública **(e,n)**, temos que encontrar um par de números primos **p** e **q** que se situal entre 2^512 e 2^513 e que o seu produto dá origem a **n**: p * q = n .

```py
def find_p_q(n):
    for p in sympy.primerange(2**512, 2**513):
        if n % p == 0:
            q = n // p
            if miller_rabin(q):
                return p,q
    return None, None
```

Algoritmo de miller_rabin foi usado para verificar o quão próximo um número é primo, quanto maior o k, mais precisão a função tem para calcular números primos grandes.
```py
def miller_rabin(number, k=5):
    if number <= 1:
        return False
    if number == 2 or number == 3:
        return True
    if number % 2 == 0:
        return False

    r, odd_part = 0, number - 1
    while odd_part % 2 == 0:
        r += 1
        odd_part //= 2


    for _ in range(k):
        witness = random.randint(2, number - 2)
        x = pow(witness, odd_part, number)

        if x == 1 or x == number - 1:
            continue

        for _ in range(r - 1):
            x = pow(x, 2, number)
            if x == number - 1:
                break
        else:
            return False  

    return True  
```

Depois, usamos esse par **p**, **q** para calcular o fi do **n** : **(p-1)(q-1)** .

Como a expressão para calcular a chave privada **d** é e*d mod fi(n) fizemos o inverso disso para calcular o **d**.
```py
def find_d(e, phi_n):
    d = sympy.mod_inverse(e, phi_n)
    return d   
```

Com a chave privada conseguimos decifrar a partir do python, com a função dec(), o ciphertext que estava codificado usando algoritmo de RSA.

### Passo 2 - Aceder o servidor através de netcat:

```c
nc ctf-fsi.fe.up.pt 6004
```
Reparamos nos seguintes dados:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1184634835948535911/image.png?ex=658cb02a&is=657a3b2a&hm=6092622a7f34d8a840afdcd5036c4af163f0f80e5cd9b953066315b0d5338140&)

### Passo 3 - Forjar o script em python:

Editamos o python da seguinte maneira:
```py
# Python Module ciphersuite
import os
import sys
import sympy
import random
from binascii import hexlify, unhexlify
import base64


def miller_rabin(number, k=5):
    if number <= 1:
        return False
    if number == 2 or number == 3:
        return True
    if number % 2 == 0:
        return False

    r, odd_part = 0, number - 1
    while odd_part % 2 == 0:
        r += 1
        odd_part //= 2


    for _ in range(k):
        witness = random.randint(2, number - 2)
        x = pow(witness, odd_part, number)

        if x == 1 or x == number - 1:
            continue

        for _ in range(r - 1):
            x = pow(x, 2, number)
            if x == number - 1:
                break
        else:
            return False  

    return True  



def enc(x, e, n):
    int_x = int.from_bytes(x, "little")
    y = pow(int_x,e,n)
    return hexlify(y.to_bytes(256, 'little'))

def dec(y, d, n):
    int_y = int.from_bytes(unhexlify(y), "little")
    x = pow(int_y,d,n)
    return x.to_bytes(256, 'little')
    
def find_d(e, phi_n):
    d = sympy.mod_inverse(e, phi_n)
    return d   
def find_p_q(n):
    for p in sympy.primerange(2**512, 2**513):
        if n % p == 0:
            q = n // p
            if miller_rabin(q):
                return p,q
    return None, None

n = 359538626972463181545861038157804946723595395788461314546860162315465351611001926265416954644815072042240227759742786715317579537628833244985694861278989853183542398728530948752173871943025756711047464100821087748836169998470243979216842437028866375662823190225125262695693690175324902816214496272114328423143
e = 65537
ciphertext = unhexlify('3163326362333033623561633535623032656262626462303138616138643163313362326363333762353065346333613130616262646531383939336437376566356166636363626532333634656536363263323631353638336331623739666262633165633333623365653065343562626336353137376637613431666137353932366139323630386365366561623733636162396439613434396530336235353132323433396363356264343630383863656363613166636430623734373534336336363936383364663837393162633732396636663261393163376638613761663231393931633233333766666237653765393238336564363032356130303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030')
p,q = find_p_q(n)
euler_totient_n = (p-1)*(q-1)
d = find_d(e,euler_totient_n)

decoded_message = dec(ciphertext,d,n)
message =decoded_message.decode('latin-1')
print("p:",p,"\n"," q:",q,"\n","euler_n:",euler_totient_n,"\n")
print("Public parameters -- \ne: ", e, "\nn: ", n,"\n")
print("d:",d,"\n")
print("decoded_message:\n",decoded_message,"\n")
print("Message:", message)


sys.stdout.flush()
``` 

#### Resultados

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1184637550669856888/image.png?ex=658cb2b1&is=657a3db1&hm=eb6564e169cd45e453916e69542f633a0c366886a437f35bc51d73226d4de4aa&)







