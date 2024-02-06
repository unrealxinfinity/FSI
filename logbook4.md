# Trabalho realizado na Semanas #4

## Task 1

- Experimentação com os comandos da shell sobre variáveis ambiente.


## Task 2


- Primeiro compilamos o ficheiro com nome de 'myprintenv.c' com o printenv() do processo filho comentado e colocamos o output desse programa num ficheiro. Depois comentamos o printenv() do processo pai , descomentando o do processo filho, compilamos e colocamos o output noutro ficheiro e comparamos os 2 ficheiros;
- Concluímos que o processo filho herda as variáveis ambiente do processo pai em default.


## Task 3

- Compilamos o ficheiro 'myenv.c' 2 vezes com diferença de que uma vez a função execve() tinha o NULL no 3º argumento e noutra vez a função tinha 'environ' no mesmo argumento;
- Colocamos os outputs em 2 ficheiros distintos e concluímos que o 3º argumento deixa que o processo filho / comando executado pelo execve consegue herdar ou não as variáveis ambiente dependendo do 3º argumento do execve. 


## Task 4

- Criamos um ficheiro e copiamos o pedaço de codigo para este;
- Compilamos e executamos;
- Concluímos este consegue passar as variáveis ambiente para o processo executado pois system() usa execl() que por sua vez utiliza execve na sua implementação e passa como 3º argumento 'environ'.

## Task 5

- Criamos um ficheiro para colocar o pedaço de código que mostra output das variáveis globais;
- Compilamos o programa e mudamos os privilégios deste e colocamos o SET_UID nele usando chmod() e alteramos ownership dele usando comando chown() para root.
- Alteramos 3 variáveis ambiente: PATH, LD_LIBRARY_PATH e uma criada por nós mesmos;
- Colocamos o resultado do printenv() para um ficheiro, colocamos o output do programa que tem SET_UID noutro ficheiro, comparamos os outputs e descobrimos que o programa SET_UID esconde variáveis como LD_LIBRARY_PATH.
- Concluímos que o sistema operativo tem algumas variáveis ambiente protegidas de alteração por outros processos que tem nível elevado de privilégio.


## Task 6 
- Compilamos o programa dado pelo exercicio;
- Mudamos os privilégios e colocamos o programa a SET_UID;
- Mudamos a shell para zshell;
- Adicionamos uma PATH variable para pasta home usando comando export : 'PATH=/home/seed:$PATH';
- dentro da pasta home , criamos um programa chamado 'ls' que finje ser programa malicioso;
- Verificamos que o programa vítima dado pelo exercicio corre o programa malicioso pois usa comando system() que herda as variáveis de ambiente do processo da shell dado pelo usuário, e ao alterar a variável para um local designado pelo atacante este consegue correr todo programa malicioso que tenha nome de 'ls' pois o programa vítima chama system com uma localização relativa e não absoluta, o que da liberdade ao atacante;
- Concluímos que o facto de correr system numa aplicação que tenha SET_UID ativado é perigoso. 
