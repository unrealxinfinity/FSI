## Desafio 1:
- ### Entender as restrições da memória e o seu funcionamento ao usar o comando checksec;
- ### Verificar main.c e reparar que quando faz scanf não existe nenhum controlo de input e pode ser induzido um buffer overflow ao escrever mais de 32 chars;
![img1](https://cdn.discordapp.com/attachments/1153998326274994216/1162165874589966366/image.png?ex=653af250&is=65287d50&hm=c5471992529eae3f7885dd3f8bc6575f4dad584fabddff45272e56ab1b5dc75c&)
- ###  Como o array meme_file com o nome do ficheiro a executar para obter a flag está inicializado em cima do buffer na stack, ao escrever 32 chars qualquer + 8 chars com nome do ficheiro com a flag,flag.txt, esses 8 bytes sao escritos por cima dos endereços que meme_file está entre, acedendo assim a flag.txt.
- ### Assim obtemos a flag.
![img2](https://cdn.discordapp.com/attachments/1153998326274994216/1162165874275385444/image.png?ex=653af250&is=65287d50&hm=cdda6ecbd357eb21c11435a3ded5130dbe662f0ae830cdb7a5756f70bc729cc4&)
- #### Input usado : aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaflag.txt

## Desafio 2:
- ### Entender qual a mudança no main.c para mitigar a falha do buffer overflow;
![img3](https://cdn.discordapp.com/attachments/1153998326274994216/1162165874942292049/image.png?ex=653af250&is=65287d50&hm=c034a72509479e85731d0cf815c4559d523a5c0371437e6450637907d6a84f9e&)
- ### Entender que tendo endereço/valor certo no buffer val[] faz com que o programa possa executar codigo para ler o ficheiro mem.txt que, substituindo no buffer meme_file[] para flag.txt podemos ter acesso a flag;
- ### Para fazer as substituições, aproveitamos o buffer overflow para colocar la os conteúdos pertinentes usando o código no python, nomeadamente r.sendline para enviar inputs: primeiros 32 chars preenchidos com coisas aleatórias + 4 chars com "\x24\x23\xfc\xfe" que o python transforma em 4 chars de hexadecimal 0xfefc2324 + 8 bytes com o nome do ficheiro flag.txt;
![img4](https://cdn.discordapp.com/attachments/1153998326274994216/1162165875277832362/image.png?ex=653af250&is=65287d50&hm=a0a80a50c1e3aecde8e9dfef1278d5a805e5e1c7a430e92e705a0916b56c3d48&)
- ### Assim obtemos a flag no output do terminal.
![img5](https://media.discordapp.net/attachments/1153998326274994216/1162165875579813938/image.png?ex=653af250&is=65287d50&hm=c294d15a93ff0f735cadd31832bf2a1393c89e5f61c83a455cfe3efa64795b6d&=)
- #### Input usado : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x24\x23\xfc\xfeflag.txt


