### Desafio : 

Para resolver o desafio começamos por analisar o codigo fornecido denominado por index.php. Aqui reparamos que havia uma vulnerabilidade que podia levar a um ataque do tipo SQL Injection. 
Essa vulnerabilidade provinha das strings não sanatizadas.

#### Pergunta 1 : Que query SQL é executada para cada tentativa de login?

A query utilizada para cada tentativa de login é : ```sql $query = "SELECT username FROM user WHERE username = '".$username."' AND password = '".$password."'"; ```


#### Pergunta 2 : Que input podes manipular para usurpar a query? Que caracteres especiais utilizaste e porquê?

O input que podemos utilizar para usupar a query é o seguinte: admin' -- . Ao utilizar este comando, estamos a comentar todo o codigo que vem após o username. Ao comentar, o sistema não conseguirá verificar o campo 'password'.

#### Pergunta 3 : Qual query SQL é efetivamente executada com a tua tentativa de login maliciosa? Porque é que essa query te permite fazer login?

A query executada é a seguinte : ```sql $query = "SELECT username FROM user WHERE username = '".$username." ``` . Este codigo como referido anteriormente, não irá verificar a password associada ao nosso user, dessa forma qualquer password será válida e todo o acesso é permitido. 


#### Implementação no Servidor: 

##### Passo 1: 

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176845606434643978/image.png?ex=657059e1&is=655de4e1&hm=90879984802c54f8cc29f46fd7eabd32e7e9f9f08323f5da7e818a99dcc64464&)


##### Passo 2:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1176845643050926120/image.png?ex=657059ea&is=655de4ea&hm=54d7b80f6b0a60e774f929f8607e34c0dea39ba16886bddde8ab09f9692a39c7&)

