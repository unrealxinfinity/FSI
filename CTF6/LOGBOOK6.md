# CTF6 (XSS + CSRF)

## Objetivo

Explorar injeção de código e o contexto de execução de diferentes páginas em segurança web

## Resolução

### Conexão

- Comçamos por estabelecer uma ligação ao servidor ctf-fsi.fe.up.pt da porta 5004.   

### Descrição

- Nesta porta temos acesso a um form onde podemos pedir acesso a uma flag e o id da request desta:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1164155871954223135/image.png?ex=65422fa4&is=652fbaa4&hm=3aa2bd13d09fdde8211a2f04c7a01a5dcf846d0f6a65b6bc5e79f89156eac4d3&)

- Ao clicar no butão de submit, somos dirigidos para uma página de espera pela validação desta request feita anteriormente:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1164156252872507442/image.png?ex=65422fff&is=652fbaff&hm=0d26331da65f0b08d28c389f26946d26bb798185ac682413bf42d02dbdade779&)

- Também temos acesso à página do administrador que controla se as requests são válidas para a obtenção da flag:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1164156329359843368/image.png?ex=65423011&is=652fbb11&hm=de88ca218479cbb5d7ca90f488c4f0f4ab39c57d2d3ede5d7eaf51364c3a5bbd&)

Nota: Os botões nesta página encontram-se desativados e a porta passou de 5004 para 5005.

### Vulnerabilidade 

- Começamos por procurar vulnerabilidades neste sistema, analisando em pormenor o form da main page de porta 5004. 


- Reparamos que esse form não tinha proteção contra ataques de XSS:


![img](https://cdn.discordapp.com/attachments/1153998326274994216/1164157466540519424/image.png?ex=65423121&is=652fbc21&hm=0e65195f09acff31911697a0b95eaf2bbd054e2441bbe4eebcb5d4d5d8cb46dd&)

- Ao escrever e submeter esse form, o resultado deste aparece na página de espera:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1164157584694067311/image.png?ex=6542313d&is=652fbc3d&hm=1556d0a2618f6f175c2fddb1ff21538c0ab367cc01291c289a85b5b5a4291960&)

- Ao colocar código HTML e JS no form, conseguimos criar código maligno com o propósito de forçar o administrador a validar-nos a request da flag:


![img](https://cdn.discordapp.com/attachments/1153998326274994216/1166097741773029520/image.png?ex=65494026&is=6536cb26&hm=4da151ecaa27bb4a2a186f77c6dbaf6f82e43f8c551027fe49d86e1c0a148135&)

Nota: Definimos um form com action a direcionar para a página de envio da flag com o id igual ao já mostrado e ao adicionar um script que submete o form, podemos forçar este a correr o conteúdo dessa página.

- O path do action leva-nos a uma página que em princípio só o administrador do website teria acesso. Este processo chama-se de CSRF. O código desta página é executado
quando o utilizador clicar no input de tipo submit criado por nós. No entanto, é necessário também incluir um script de java script que vai submeter o código na página de espera:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1166100600799367329/image.png?ex=654942d0&is=6536cdd0&hm=d6b358f274b70c985320e2b926b802a0aeaa633d4162fc2ecd33d2867d74f4ae&)

- Ao submeter este form, clicando assim no botão de submit, enviamos este código maligno para a página de espera onde somos rapidamente redirecionados e conseguimos ver o nosso código de html representado visualmente em forma de botão:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1166341524288712724/image.png?ex=654a2330&is=6537ae30&hm=14aaab1c9748c3677c120a627ef85ee06aff588b01face39f061c6f903b476a4&)

- Como a página onde nos encontramos agora efetua refreshes 5 em 5 segundo, então temos de pressionar o botão com rapidez, senão já não dará para efetuar este ataque


- Por fim, após clicar no butão, esperamos pelo próximo refresh da página, entretanto o código da página que dá approve da request estará a ser executado e a flag será demonstrada após estes 5 segundos:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1164152216211439667/Captura_de_ecra_de_2023-10-18_11-44-54.png?ex=65422c3d&is=652fb73d&hm=d88d21c6f174327f7257cefe2f6b02aedb22e98e8d247516df52728f471afd4c&)




