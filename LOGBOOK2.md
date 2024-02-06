# Trabalho realizado nas Semanas #2 e #3

## Identificação

- A vulnerabilidade [CVE-2023-42454](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-42454) afeta as versões anteriores a 0.11.1 do SQLpage , uma ferramenta de criação de aplicativos web que utiliza apenas SQL;
- A vulnerabilidade está relacionada com a string de conexão da base de dados especificada no arquivo de configuração sqlpage/sqlpage.json que era então pública; 
- A partir da vulnerabilidade o atacante passa então a ter acesso a toda a base de dados da vítima;

## Catalogação

- Publicado em 2023-09-18 às 22:15:48, atualizado em 2023-09-19 às 03:37:19, Origem - Github., Inc;
- A vulnerabilidade foi abordada e corrigida na Versão 0.11.1;
- De acordo com a [NVD](https://nvd.nist.gov/vuln/detail/CVE-2023-42454) o base score é 10.0;

## Exploit

- A partir do browser é possível baixar os ficheiros contidos na pasta SQLpage;
- A exploração é bem-sucedida quando a string de conexão da base de dados é acessível no arquivo de configuração sqlpage/sqlpage.json;
- Essas informações são usadas pelo atacante para se conectar diretamente á base de dados da vítima;
- Com a conexão feita o atacante terá acesso a todas as informações contidas na base de dados da vítima; 


## Ataques

- Até ao momento não foram registados quaisquer ataques;
- Um possível ataque permitiria ao invasor ter acesso a todos os dados da base de dados;
- Como forma de mitigação, foi aconselhado a todos os utilizadores a atualizarem a versão de SQLpage;
- Usuário no github com username de olivierauverlot descobriu a falha de segurança;

## Fontes
- https://nvd.nist.gov/vuln/detail/CVE-2023-42454
- https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-42454;
- https://github.com/lovasoa/SQLpage/issues/89#issue-1900340633;
