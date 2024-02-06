## Public-Key Infrastructure (PKI)

### Overview

A PKI é uma estrutura usada para encriptar e desencriptar mensagens mandadas entre utilizadores através da internet, ou pedidos de http. Já vimos no lab anterior (Secret-Key Encription) que ter uma chave só não é ideal, pois é necessário fazer duas ações, encriptação e desencriptação, e estar a colocar essa responsabilidade toda numa só chave é demasiado vulnerável.

Nesta abordagem de PKI, conseguimos usar uma chave para encriptação e outra chave para desencriptação, sendo que a para encriptar se chama de public key e a de desencriptar de private key. Cada utilizador tem uma chave pública e privada, e, tal como o nome indica, a chave privada só é disponibilizada ao próprio utilizador, enquanto que a chave pública é disponibilizada a todos os utilizadores que querem mandar uma mensagem com o próprio utilizador. Na eventualidade de um utilizador querer mandar uma mensagem a outro utilizador, ele irá pedir pela sua chave pública para encriptar a mensagem a enviar e o recetor da mensagem usa a sua chave privada para desencriptar o conteúdo tal como mostra a imagem seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180915897767047289/image.png?ex=657f28a2&is=656cb3a2&hm=260fea2eb6ebae475b53a53921234bb7d39cffeafbd4f61ae5bfda50484ee5fe&)




No entanto, um dos grandes problemas desta abordagem é o ataque do Man-in-the-Middle. Este ataque consiste em uma pessoa que não deveria estar envolvida no envio ou receção da mensagem entre utilizadores, no entanto ela consegue-se meter no meio da ação, intercetando assim a mensagem a enviar e ainda podendo mandar uma mensagem própria dele ao recetor da mensagem original, passando-se assim de despercebido:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180916753363107920/image.png?ex=657f296e&is=656cb46e&hm=195a65cdd0a2e50c6bf02642d1665bbc3fe2d327d31944f20b147b83193c065c&)

Neste lab vamos ver como fazer este ataque e como mitigá-lo

### Lab Environment

Para inicializar este lab tivemos que descomprimir um zip dado pelo seed labs:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179379131415265391/image.png?ex=65799168&is=65671c68&hm=41ec29567857fb3609f5319323b4c0cad296e55e32e0d4310bfb26035cf9c4b8&)


Também tivemos que verificar se o ficheiro de hosts encontra-se atualizado para este lab. Podemos verificar isso utilizando o seguinte comando:

```c
$ cat /etc/hosts
```

Este comando originou o seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179386174628974612/image.png?ex=657997f7&is=656722f7&hm=9483bf04b16c631e21040d8f41c207c4dfc9555181cf19d75a0c172c7b548742&)


Como podemos verificar, não existe uma coneção para o site a ser usado neste lab, logo ao usar um comando para editar o ficheiro, podemos inseri-la nós próprios:

```c
$ sudo nano /etc/hosts
```

Escrevendo assim no ficheiro 10.9.0.80   www.bank32.com, que são os valores que no seed labs dizem para por nesse ficheiro:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179386269168566272/image.png?ex=6579980e&is=6567230e&hm=730a0745eb36e09fbea43acbd6d2a41cbca5f348e0897e36ff8860e46066f676&)




### Lab Tasks

#### Task 1: Becoming a Certificate Authority (CA)

A forma para proteger de ataques de Man-in-the-Middle é usando autoridades certificadoras. Estas servem de uma confirmação para poder efetuar a mensagem a ser enviada e recebida. Devido ao facto que estas autoridades são muito conhecidas, são praticamente impossíveis de replicar por um utilizador malicioso, logo se a mensagem que for enviada for certificada por uma destas autoridades, em princípio assume-se um modelo de confiança que a mensagem chegará ou será recebida pelo destinatário em segurança e que ninguém a irá intercetar a meio:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180919581141913650/image.png?ex=657f2c10&is=656cb710&hm=b87e23aac4285feade9ea7bb54d9786997517a2b4e805330b78b7264e8344a56&)



Nesta task vamos tentar criar uma autoridade certificadora, que vamos assumir, por simplicidade, que é uma autoridade fiel, pois somos nós que as criamos.

Primeiro temos que fazer umas configurações. Temos que criar um ficheiro openssl.conf e depois copiar o conteúdo do url /usr/lib/ssl/openssl.cnf para esse ficheiro, comenentando também a linha de unique_subject onde diz CA_default:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180955802417254500/image.png?ex=657f4dcc&is=656cd8cc&hm=433a9c800ef4476f66849aa7c316864909fb9598f03baa83e4abc67df5e52c42&)

Como podemos ver, na imagem anterior é indicado as diretorias e ficheiros default que o sistema deve ter para estas configurações. Para ter este sistema default temos que fazer os seguintes comandos:

```c
$ mkdir demoCA
$ cd demoCA
$ mkdir certs crl newcerts
$ echo 1000 > serial
$ touch index.txt
```

Após estes comandos também passamos o ficheiro de configuração openssl.cnf para a diretoria demoCA obtendo assim estes resultados:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179386374311399444/image.png?ex=65799827&is=65672327&hm=165dbcff9bb4886ff36aefe11d08ed1ce1cb0f003d98e2a6d05a1404b6b7c04d&)

Dentro da diretoria demoCA:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179386431358124082/image.png?ex=65799835&is=65672335&hm=62c990981ee1d03385575e3224e85244edff3861dc11efcc59a24fc605796a18&)


Agora para gerar um CA self-signed usamos o seguinte comando:

```c
$ openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -keyout ca.key -out ca.crt
```

Depois preenchemos os parâmetros com os seguintes dados:

- pass phrase: Gatosecaes123;
- Country Name: PT;
- State or Province Name: Porto;
- Locality Name: FEUP;
- Organization Name: FEUP;
- Organizational Unit Name: FSIl03g02
- Common Name: jomi
- Email Address: jomi@gmail.com

Para a geração das chaves é utilizado um algoritmo de RSA, que essencialmente utiliza dois números primos muito grandes e multiplica-os para formar um módulo em que vai ser encriptado/desencriptado os dados.

Usando os seguintes comandos podemos ver o conteúdo do certificado e da chave RSA, respetivamente:

```c
$ openssl x509 -in ca.crt -text -noout
$ openssl rsa -in ca.key -text -noout
```


Através destes comandos conseguimos ver e confirmar informações importantes para nós, tal como:

##### What part of the certificate indicates this is a CA's certificate?

A parte do certificado que indica que é um certificado CA encontra-se no conteúdo do certificado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179388817816432810/image.png?ex=65799a6e&is=6567256e&hm=20e1a36ef588051aeeaf4d9065fbdb4a8b5bc7d3f582f70a0deeef4a1526f973&)

Como diz que CA:TRUE então sabemos que o certificado é um CA certificado.

##### What part of the certificate indicates this is a self-signed certificate?

A parte do certificado que indica que é um certificado self-signed encontra-se no conteúdo do certificado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179388738665717840/image.png?ex=65799a5b&is=6567255b&hm=01e28c3fb9aad411092d5e322558d49f0f8c7840724956f91ea6c8ebc05b6d76&)


Como o identificador 'Issuer' é igual ao 'Subject', então sabemos que o certificado é da mesma entidade.

##### What are the values of the public exponent e, private exponent d, modulus n and two secret numbers p and q such that n=pq in the RSA algorithm?

Estes valores encontram-se presente na chave de RSA.

Valor do expoente público e:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179391097072529478/image.png?ex=65799c8d&is=6567278d&hm=0ed55bac88bea391a71f0415b00225fc1f49750345b4d48477714dbba7edbc7c&)

Valor do expoente privado d:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179391176940466236/image.png?ex=65799ca0&is=656727a0&hm=438fef7874003a451f28d918357cd1ac9c3069b0caaee37c3f56f204cca9cda5&)

Valor do modulus n:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179391305978220564/image.png?ex=65799cbf&is=656727bf&hm=86b148e9d048c8d2fd5e0ec0e2cfeeed0e6af34de2afdb98c7fd909fca99d685&)

Valor dos dois números secretos p e q tal que n=pq:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1179389150915461140/image.png?ex=65799abd&is=656725bd&hm=9bf106c236687cb4eacfa676c5146a255d9f37cda296d57efa2fabe6682da7a7&)

#### Task 2: Generating a Certificate Request for Your Web Server

O servidor web que vamos usar chama-se www.bank32.com.

Primeiro precisamos de mandar um certificado de chave pública para o servidor a partir do nosso CA. Por isso tem de ser gerado um Certificate Signing Request (CSR) e também vamos incluir nomes alternativos, pois vários sites da web têm url differentes, logo vamos inclui-los. O comando para gerar o CSR e incluir urls differentes é o seguinte:

```c
openssl req -newkey rsa:2048 -sha256 -keyout server.key -out server.csr -subj "/CN=www.bank32.com/O=Bank32 Inc./C=US"  passout pass:Gatosecaes123. -addext "subjectAltName = DNS:www.bank32.com, DNS:www.bank32A.com, DNS:www.bank32B.com"
```

#### Task 3: Generating a Certificate for your Server

Vamos fazer o nosso CA gerar certificados. Para fazer isto, primeiro é importante comentar uma linha no ficheiro openssl.cnf:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180958835901485256/image.png?ex=657f509f&is=656cdb9f&hm=f06e2465ebd20bfaff263e62f80c7bf270b2e5681724e1005bf3a8c4ea9722a8&)

Resultado final é o seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180958927727366194/image.png?ex=657f50b5&is=656cdbb5&hm=7b125677d43ac0c6c65dfb13ff16405c9bd37ddd3f2515033b36093e782c4e2a&)

Após essa pequena configuração utilizamos o seguinte comando para gerar um certificado a partir do nosso CA:

```c
$ openssl ca -config openssl.cnf -policy policy_anything -md sha256 -days 3650 -in server.csr -out server.crt -batch -cert ca.crt -keyfile ca.key
```

Gerando assim o seguinte certificado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180960020695896094/image.png?ex=657f51ba&is=656cdcba&hm=36b6b6153b8b3a63bdb1461873069b74e632b25d0c40758ada5a9dd1ed60bb84&)

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180960078128496690/image.png?ex=657f51c7&is=656cdcc7&hm=b04f57ea03db43fc6a767091f9e0e5c8fbdf36eb1b4ede9c1ee456fd77583eb9&)

E podemos verificar que os 3 urls que incluímos na task anterior se encontram no certificado através do seguinte comando:

```c
$ openssl x509 -in server.crt -text -noout
```

Estando aqui os urls no certificado:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180959320578457600/image.png?ex=657f5113&is=656cdc13&hm=925424d16f8b2c616ffa8275bfba94c919e5ff05e72de6f7d9915b51fb053f97&)

#### Task 4: Deploying Certificate in an Apache-Based HTTPS Website

Na parte inicial desta task vamos colocar o site a funcionar. Para isso fomos ao ficheiro de configuração já disponibilizado pelo seed labs que se chama bank32_apache_ssl.conf:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180964763417710602/image.png?ex=657f5624&is=656ce124&hm=b357652c9aa27170b72a032b3c72961b7410fecf1527b4653d9621f344910750&)

E modificamos o path do SSLCertificateFile e o SSLCertificateKey para a diretoria volumes, também disponibilizada pelo seed labs:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180964990690275428/image.png?ex=657f565b&is=656ce15b&hm=19e6ef23f3c26845375b85c3af2c66b40664ff3eef9b58eb2a4fe4f106c72723&)

E agora tivemos que copiar para a diretoria volumes os ficheiros da key e de certificados pois o docker agora está à espera nessa diretoria por esses ficheiros nossos, e também naturalmente tivemos que modificar os nomes dos ficheiros para que fiquem iguais aos que estão no path do ficheiro de configuração anterior:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180967621202546819/image.png?ex=657f58ce&is=656ce3ce&hm=4fb997e60ab16ea93b20f852fd23e788d33a6da7e5f292a58c20f019dd21384b&)

Agora ao corrermos os comandos de build do docker e docker compose up conseguimos que o ficheiro de configuração dê atache a esses dois ficheiros que disponibilizamos.

Para efetuar os comandos, abrimos um terminal differente e fizemos o seguinte:

```c
$ dcbuild
$ dcup
```

No final deste lab, iremos usar o seguinte comando para fechar o servidor:

```c
$ dcdown
```

Ao fazer o seguinte comando:

```c
$ dockps
```

Conseguimos descobrir o estado do docker e verificamos o seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180968868550819951/image.png?ex=657f59f7&is=656ce4f7&hm=48edfdc3aa8540efbf521a9efc0ccd6d448a907d54cc80991b149de33471a599&)


Abrimos uma shell nesse container através do seguinte comando:

```c
$ docksh ad
```

E agora que já temos uma shell no container efetuamos o seguinte comando para começar o servidor Apache:

```c
# service apache2 start
```


Nota: para parar o servidor e para o reiniciar os comandos são os seguintes, respetivamente:

```c
# service apache2 stop
```

```c
# service apache2 restart
```

Depois de ter começado fomos para o browser e colocamos no url www.bank32.com se deu-nos a seguinte página:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180969287280762920/image.png?ex=657f5a5b&is=656ce55b&hm=11b6046c58b26feb81365e8e2b9c9523436132d638e8e89fc036bef595b82333&)

Mas esta página não é a correta, pois não está a usar o https está a usar http que torna o browser inseguro, pois não há uso de CA e pode acontecer ataques de Man-in-the-Middle, logo ao pormos https no início verificamos o seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180971094119809074/image.png?ex=657f5c0a&is=656ce70a&hm=0c32a7c33f05f9656daf16532da6378255a4c0ca3f523a11cf9ef7de1c90ef4c&)


No entanto, reparamos que o site não está seguro:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180971828085280849/image.png?ex=657f5cb9&is=656ce7b9&hm=41555c137153d7f05424ed950b6e4fc462cce04beaf38021d2a2d9542f854d5c&)

Isto deve-se pois o site precisa da permissão do nosso certificado, pois temos que ser nós a dar esta permissão visto que o certificado que criamos não faz parte de uma autoridade certificadora conhecida, logo o browser não tem a certeza se esse certificado é maligno ou não.


Para dar permissões ao nosso certificado temos que seguir o seguinte caminho:

Fomos para esta página através do url mostrado na seguinte imagem:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1180972815428616312/image.png?ex=657f5da4&is=656ce8a4&hm=4387d181797c2fb675322aa65730b426be2b215955b33a6bc3392a4d0354b838&)

Dirigimo-nos para Certificates->View Certificates->Authorities->Import 

Importamos o certificado que nós criamos:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1181932948837105705/image.png?ex=6582dbd6&is=657066d6&hm=149deb41c9fe930f0dd7d58efb3082173b631c3e2f02d102dfdadd64b0d8091c&)

E observamos o seguinte:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1182324321339838474/image.png?ex=65844854&is=6571d354&hm=36f519d07d2875d38335d4517389abd9bf66c4af2a358630773e136b3e078bbc&)

O servidor agora já reconhece o nosso certificado como sendo um certificado seguro e fiel.

#### Task 5: Launching a Man-In-The-Middle Attack

Para realizar esta task tendo em conta as tasks anteriores, só precisavamos de alterar o domínio que o server apache estava a apontar para www.example.com com o seguinte comando:
```c
nano bank32_apache_ssl.cnf
```
![img](https://media.discordapp.net/attachments/1153998326274994216/1182531909251780709/image.png?ex=658509a9&is=657294a9&hm=0806faadb93698bad95754cfbf224e8c4f9140be53da0ebb8a23388418c75555&=&format=webp&quality=lossless&width=1398&height=662)

E criar um endereço ip com esse mesmo dominio no ficheiro /etc/hosts do lado da vítima 
![img](https://media.discordapp.net/attachments/1153998326274994216/1182531909557948527/image.png?ex=658509a9&is=657294a9&hm=b33f6a351380f33b05e95168c40c756a0254bc7c738151406a0a321c3d7698db&=&format=webp&quality=lossless&width=1440&height=288)


Os resultados indicam que um atacante malicioso ao mudar o endereço de ip com uma determinada route para o domínio que leva à página gerida pelo servidor apache malicioso pode enganar a vitima e pode recolher informações desta.

Site antes de ser intercetado:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1182554627087016027/image.png?ex=65851ed1&is=6572a9d1&hm=39c3d68195f8e44100a2549bc164a6607676ba15184298059617ccb19f541ede&)

Site depois de ser intercetado:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1182531908702326844/image.png?ex=658509a9&is=657294a9&hm=5d7ab1e070be80cb81402d1a2023b6ebcf31fa80d0fb095964b96b8e2ba2d79f&)


#### Task 6: Launching a Man-In-The-Middle Attack with a Compromised CA

Para realizar este ataque, foi preciso simular que o atacante teve o acesso à chave privada gerada pelo certificado CA e ter em conta as tasks anteriores.

A partir dessa chave, podiamos gerar um certificado assinado pela chave privada do CA e usar esse certificado no servidor malicioso apache que interceta um domínio tornando-o malicioso.

Primeiro geramos uma certificado CA através da chave que foi comprometida do CA original para ser usado mais tarde no browser na whitelist:
```c
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 \
-keyout ca.key -out ca.crt \
-subj "/CN=www.example.com/O=FEUP/C=PT" \
-passout pass:dees
```

Foi preciso gerar um pedido de validacão para o site em questão www.example.com atraves do comando:
```c
openssl req -newkey rsa:2048 -sha256  \-keyout server.key   -out server.csr  \-subj "/CN=www.example.com/O=FEUP/C=PT" \-passout pass:dees
```
Esse pedido foi gerado para criar um certificado assinado por um certificado CA que geramos:
```c
penssl ca -config openssl.cnf -policy policy_anything \-md sha256 -days 3650 \-in server.csr -out server.crt -batch \-cert ca.crt -keyfile ca.key
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1182568854812115014/image.png?ex=65852c12&is=6572b712&hm=1f1635c0a2d031c56e93a2a300fa7cace1de8c392c3aa11af5ec92d43b3efc9a&)

Trocando o server.crt , server.key e o ca.crt pelos ficheiros na pasta certs da imagem do docker ou renomeando os e configurando o dockerfile para fazer uma imagem apropriada, conseguimos fazer com que o website seja acessivel sem ser suspicio com um certificado valido.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1182569657966796841/image.png?ex=65852cd1&is=6572b7d1&hm=ff88fd46550dc090db67bcec19defd688187a77173457cc446ab113ea1433c09&)

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1182570097039126630/image.png?ex=65852d3a&is=6572b83a&hm=7e0c8cafb396aa6f3d1d6ec3fad6ff573bf4971334a3e1aa62f51d83c9c47f0e&)

Depois de confiar o CA que geramos no brower:
![img](https://cdn.discordapp.com/attachments/1086437609615667300/1183469207841681458/image.png?ex=65887296&is=6575fd96&hm=ce49c81efb0a018b1c3e09eddc5b51176348265f199d93a58158697ef5d18365&)

Resultados:
![img](https://cdn.discordapp.com/attachments/1086437609615667300/1183469161809182720/image.png?ex=6588728b&is=6575fd8b&hm=6acde12257994abb697117e290d7adbd923947371321d682e63754873a10a4fb&)
