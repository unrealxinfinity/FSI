## Sql Injection


### Overview

Neste seed labs iremos identificar e explorar vulnerabilidades associadas à utilização de sql injection através de um input malicioso do utilizador que controlo as queries que são feitas a uma base de dados a correr num servidor.  

### Lab Environment

É disponibilizado uma aplicação web para este lab, que se encontra no url http://www.seed-server.com com um IP do container 10.9.0.5. 

Para podermos usar esta aplicação, precisamos de mapear o hostname para o IP do conatiner. Para fazer isto, primeiro verificamos no ficheiro /etc/hosts se encontra-se bem configurado. Ao fazer este comando no terminal:

```c
$ cat /etc/hosts
```

conseguimos ver o que está configurado no ficheiro:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176911269983428709/image.png?ex=65709708&is=655e2208&hm=6204ab4b6b233273098564c1cf9d9282e875959e4a5013981a4d1ffd163c5510&)

Este é o aspeto que deveria ter, para o container com IP 10.9.0.5 deveria estar connectado ao hostname www.seed-server.com. No entanto se este não for o caso então é necessário escrever neste ficheiro e colocá-lo com o hostname correto:

Este comando permite alterar o ficheiro /etc/hosts:

```c
$ sudo nano /etc/hosts
```

depois de alterar para www.seed-server.com, basta clicar em control x e clicar y, para guardar mudanças feitas ao ficheiro.




#### Container Setup and Commands

Em primeiro lugar, é necessário descomprimir o ficheiro zip associado a este lab que terá este aspeto:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176914062144176179/image.png?ex=657099a2&is=655e24a2&hm=dba0e4c856c803768d92f4961c5396e65827d1aa912d0d1983fd67a6d5c69d5b&)

Como neste lab vamos efetuar exploits a um servidor então teremos de usar, na diretoria onde se encontra o ficheiro docker-compose.yml, comandos para iniciar o servidor:

```c
$ dcbuild
$ dcup
```

Também importante notar, que no final deste lab também é sugerido terminar o servidor usando o seguinte comando no terminal:

```c
$ dcdown
```

Após ter feito dcbuild e dcup, então saberemos que a aplicação web está pronto para ser aberta quando virmos esta linha no terminal:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176915472508264528/image.png?ex=65709af2&is=655e25f2&hm=58b8f08c6d2d97f12909c6533615eafcd0a35767611a3ff75ad3ecc55a36bef6&)

Após esta linha do terminal, abrimos a aplicação web, escrevendo no url www.seed-server.com, que abriu a seguinte página:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176915892530065509/image.png?ex=65709b56&is=655e2656&hm=050ecbedcc07c85edca3e0e5cb1030cc2a59d3751a9ed5bced45f0ffff04d953&)

Agora para podermos usar comandos num container, precisamos de obter uma shell nesse container. Para fazer isto, é necessário saber o ID do container, abrindo um segundo terminal e usando o comando que verifica o estado do docker ou containers do docker conseguimos saber os IDs dos containers:

```c
dockps
```

Este comando mostrou-nos o seguinte no terminal:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176917014565113967/image.png?ex=65709c62&is=655e2762&hm=5d5ec3be0cb6097d12e4f0288054908273a6b4677af2678fbbe89f2a78901e30&)

Na imagem vemos que o servidor de sql está a correr no IP 10.9.0.6 com ID igual a f9b04df0f380, logo para obter uma shell nesse container, basta usarmos o seguinte comando:

```c
docksh f9
```

sendo que o identificador a seguir de docksh basta ser diferente do que os restantes IDs do container que já o encontra com sucesso.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176918080086081566/image.png?ex=65709d60&is=655e2860&hm=226a91c4596bb886e2145b806ba1009b821525ff77db7ac386eb8b9097943c44&)

Nota: Ao abrir o servido vai ser criada uma nova diretoria chamada mysql_data

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176918470877794386/image.png?ex=65709dbd&is=655e28bd&hm=d37d6c8c6a58b95842ef15c05d2184595b4898901cfb208a1cddc7e7087b0ca9&)

Para começar de novo este lab basta remover esta diretoria, que contém a informação das ações feitas no servidor anteriormente, utilizando o seguinte comando:

```c
$ sudo rm -rf mysql_data
```

removendo assim esta diretoria.

#### About the Web Application

Esta aplicação da web é uma simples aplicação que regista as contas de utilizadores numa companhia, sendo que também há utilizadores com permissões de administrador que tem acesso a informação e ações privilegiadas no servidor.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176919370686013440/image.png?ex=65709e94&is=655e2994&hm=395f34de181ddda0810d391c3bca3f8a8aab3541f83217758ea0f4390e2cf3ce&)


### Lab tasks

#### Task 1: Get Familiar with SQL Statements

Agora que temos uma shell no container da base de dados, podemos aceder à base de dados através deste comando:

```c
# mysql -u root -pdees
```

que indica que o user name é root e a password é dees,
esse comando levou-nos para o cliente mysql que podemos usar para interagir com a base de dados:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176920606747406426/image.png?ex=65709fba&is=655e2aba&hm=25925f6de917724813131d04e7a62e4a18e481925964c34fbed138ccce8af069&)

Agora que estamos dentro do cliente mysql, podemos criar ou dar load de uma base de dados. Neste lab, já é providenciado uma base de dados com nome sqllab_users. Para ver todas as tabelas desta base de dados, basta usar os seguintes comandos:

```c
mysql> use sqllab_users;
mysql> show tables;
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176921333603508234/image.png?ex=6570a068&is=655e2b68&hm=b0c4388e56161e0ad3580c2725ffc53d8feed9e35e8e66b63f3dec90093f42f2&)

Para ver o que se encontra dentro da tabela credentials, basta efetuar uma simples query de sql:

```c
mysql> SELECT * FROM credential;
```

Para ver a informação da conta do utilizador com nome 'Alice', basta usar o select anterior mas com uma clausula where para especificar o nome 'Alice':

```c
mysql> SELECT * FROM credential WHERE Name = 'Alice';
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176922316496703641/image.png?ex=6570a152&is=655e2c52&hm=9416c16ee76306caaa5aef1ce3b7d040be9c7a3de618c6df19c8566daebb1ef3&)

#### Task 2: SQL Injection Attack on SELECT Statement

SQL injection é quando um user através de um input, normalmente num forms de html, consegue alterar uma query da base de dados, de forma a ver ou modificar conteúdos com que 
não deveria ter permissões.

Na diretoria Code disponibilizado pelo zip, encontra-se o código usado para efetuar queries à base de dados. No ficheiro unsafe_home.php vemos o código usado para a query de login na aplicação www.seed-server.com

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176923847916789830/image.png?ex=6570a2bf&is=655e2dbf&hm=312b4a6fde00a170e2e0091f1aa2fd520f0238f89b61fcf47890bacef1a0d454&)

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176924626752909374/image.png?ex=6570a379&is=655e2e79&hm=2dc5555e7a5b5dbac2b6ee05451e06ddbe97446d583196995e59db549b9601ff&)

Neste códigos vemos que uma query de SELECT está a ser feita para comparar o user name e a password da base de dados com os inputs do user. Também verificamos, que se o user name for o do administrador e a password for a correta então irá ser mostrado todas as informações de todos os utilizadores da aplicação.

##### Task 2.1: SQL Injection Attack from webpage

A tarfefa da Task 2 é fazer login na aplicação web www.seed-server.com como o utilizador com permissões de administrador. O utilizador com permissões de administrador é o que tem nome 'admin'.

Para efetuar este ataque teremos que colocar como input no user name o nome 'admin' visto que só com este nome que vamos ter acesso a todas as informações dos utilizadores da aplicação web:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176926846143696966/image.png?ex=6570a58a&is=655e308a&hm=bb6f80b285444718c30486a60f428f7c155c3fdbad9d66e553eb44f00cb48a89&)

agora se repararmos na query a ser feita à base de dados, vemos que, depois de verificar se o nome do utilizador é igual ao input dado do user name, irá verificar se a password é correta à dada como input do utilizador. No entanto se no input de utilizador adicionarmos uma película e um # de forma a fechar o input de username de forma a que seja igual a 'admin' e logo a seguir usar o hashtag para comentar a condição de verificação da password que vem após a verificação do user name. Desta forma, conseguimos passar a autenticação da password e entrar na conta de admin diretamente.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176927526229131294/image.png?ex=6570a62c&is=655e312c&hm=9c780fa68b94de4bba865d766462dcd4d28bc82307c8e079e5040b79c48c72ad&)

Com isto, verificamos que conseguimos autenticarmo-nos na conta de administrador e verificamos todos os dados das contas dos utilizadores desta aplicação:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176927773592399963/image.png?ex=6570a667&is=655e3167&hm=7c9235392f85c815050b8706c257c50673545ec3f52232d8a6f951a3d9acfe13&)

##### Task 2.2: SQL Injection Attack from command line

Nesta subtask, temos que fazer o mesmo que a subtask 2.1, no entanto, temos que user o terminal para fazer comandos para o servidor. Usando o commando curl conseguimos mandar uma http request, mas temos que ter em atenção que a seguir do comando curl temos que mandar o url da request, ou seja o url terá argumentos passados que são os inputs do usuário no forms. Como o & e outros simbolos do url podem ser interpretados de forma diferente no terminal, então o url tem de ser encapsulado de películas:

```c
$ curl 'www.seed-server.com/unsafe_home.php?username=admin'#&Password='
```

No entanto como já encobrimos o url do curl com películas, então quando ele ler a outra película após o username, ele irá interpretar mal a query e vai acabar o curl aí. Por isso para substituír estes carateres especiais, usamos o número ascii deles em hexadecimal com um '%' atrás dele. Fizemos o mesmo para o '#':

```c
$ curl 'www.seed-server.com/unsafe_home.php?username=admin%27%23&Password='
```

E esse comando irá fazer o http request corretamente e temos acesso novamente à conta de admin:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176965453973823589/image.png?ex=6570c97f&is=655e547f&hm=b982adbcc1ddc933c20df760d2473e37545cd5949dd0d4617597906a30d190c8&)

##### Task 2.3: Append a new SQL Statement

Nesta subtask queremos tentar correr dois comandos de sql através de um só:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176976492106874950/image.png?ex=6570d3c6&is=655e5ec6&hm=eef8024b2df99fdc852e834b164adc74021ee13b675083be615570a78444a5f2&)

O resultado foi:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176976638303539361/image.png?ex=6570d3e9&is=655e5ee9&hm=921d9061d10605d7b49f263a74b0be69f45f82431b23fb274dc20a686d408912&)

Como vemos, dá um erro de syntax. No entanto, aonde é que está o erro de syntax? Porque é que não conseguimos correr dois comandos de SQL?

Se tentarmos correr os comandos através do cliente mysql:

```sql
mysql> select * from credential where Name='admin';select 5;#and Password=;
```

O resultado desta query:


![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176978108000239686/image.png?ex=6570d548&is=655e6048&hm=8d9ab3bf11192d0648eda7191ce9bbb7933ceb675a15bcab331e22b3ea5acfc7&)

Como vemos no cliente mysql funcionou, mas na request http da aplicação web deu um erro de syntax. Então porque é que age de formas diferentes?

A resposta está no código presente em unsafe_home.php, utilizado para fazer a query à base de dados:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176978580639580291/image.png?ex=6570d5b8&is=655e60b8&hm=546f53500bf0bbabddb6eadc8fc775ed29a8034d1078a0088a6fcf44fe152f0a&)


Se repararmos, é usado $conn->query($sql). O método query é um método que só permite um statement de sql a ser feito numa request, ou seja, ele verifica se a query, que está a ser feita, inclui mais do que um comando de sql (select, update, delete), e se incluir dará um erro de syntax:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176979427045945384/image.png?ex=6570d682&is=655e6182&hm=347a2dda8f5b266ed6c95a8b672f0822436a2e4096533806c0760d7c376ff99f&)

#### Task 3: SQL Injection Attack on UPDATE Statement

Nesta task iremos ver como o comando de sql UPDATE é muito perigoso, pois consegue modificar resultados da base de dados. Este comando é algo que SQL Injection procura usar para tornar os seus ataques mais perigosos. Nesta task iremos usá-lo para modificar atributos da tabela credential, e este comando é usado na página de editar de perfil a que qualquer usuário tem acesso:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176992124458254396/image.png?ex=6570e255&is=655e6d55&hm=d65e63b2424dd4e3199d6009f046582a1640dd651b3774025f5dc6f820d04095&)

O código desta página encontra-se no ficheiro unsafe_edit_backend.php e a ação de atualizar os dados do usuário encontra-se nesta imagem:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176993114183958539/image.png?ex=6570e341&is=655e6e41&hm=18c467c6445572cac6caebb7b8a87684de2b0e67cec310f3bbd5a66a3e0b24f9&)


##### Task 3.1: Modify your own salary


Nesta subtask, vamos tentar usar SQL Injection com o uso do commando UPDATE já usado na ação de editar perfil, para atualizar o salário de um usuário. Neste caso iremos usar o usuário Alice, que não tem permissões de administrador, logo este ataque provoca muitos danos, pois editar o próprio salário não é algo que qualquer usuário deveria ter acesso.

Basta simplesmente fazer este ataque:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176994062822285312/image.png?ex=6570e424&is=655e6f24&hm=e2a200db8d30a7abb0b5cd0d2c4fa98fd26702b33a7675e5979e02e49fea2e8b&)

Que já conseguimos alterar o salário para um valor mais elevado e descartar o resto dos updates através do '#'.

Antes de mudar o salário:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176994384336662528/image.png?ex=6570e470&is=655e6f70&hm=19b91b242571f2024ebd2b62dd0d111cc4ac0dfb2a95d9f3688fc76ffd9b7153&)

Depois de mudar o salário:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176994533234450554/image.png?ex=6570e494&is=655e6f94&hm=c880a483ce556741ed058b8352b069e73c50e69f23f2e7756af6554569123480&)

Como observado, o salário foi de facto modificado com sucesso.

##### Task 3.2: Modify other people's salary

Nesta subtask queremos modificar, usando o mesmo cenário, o salário de outro usuário através da página de editar perfil da Alice. Vamos tentar modificar o salário do Boby para 1 usando este input no NickName da Alice:

```c
Alice',salary=1 where Name = 'Boby';#
```

Conseguimos modificar o salário do Boby para 1.

Credenciais do Boby antes do ataque:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176996212356947968/image.png?ex=6570e624&is=655e7124&hm=cdfd4b11f61a0811aac49ebf4a7f14d8c2d38b68366411d0f85fd5b3b09841e4&)

Credenciais do Boby após o ataque:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176996420012752896/image.png?ex=6570e656&is=655e7156&hm=5af1bc3009e9dc401ea8e034da29cc660d05f95ce4897fae34c3d13070c40284&)

##### Task 3.3: Modify other people's password

Nesta subtask, na página de editar perfil da Alice, vamos tentar modificar a password do Boby. Repara-se que não é tão simples como na subtask 3.2, pois as passwords são guardadas na base de dados hashed, neste caso através do método sha1(), que leva como input a password e faz hash desta. No entanto, o método usado é relativamente o mesmo, mas com esta pequena exceção da password. Visto isto, o input usado foi no input de NickName e foi o seguinte:

```c
Alice',password=sha1('1') where Name = 'Boby';#
```

Ao fazer login na página como Boby, colocamos no input da password o valor 1:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1177000312335249448/image.png?ex=6570e9f6&is=655e74f6&hm=c2743e8f1afbbafbb09ace8c63dbec65bb43d3c126e7028753401c30b361989f&)

E conseguimos entrar na página do Boby:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1177000456648675338/image.png?ex=6570ea18&is=655e7518&hm=94e1919030d6a74faaa0be78822b1419b1615bea6378b575603d20706a3503f7&)



