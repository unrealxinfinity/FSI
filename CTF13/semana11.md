# CTF SEMANA 13 (Find-my-TLS)

## TLS


O TLS (Transport Layer Security) é um protocolo de segurança que viabiliza a comunicação segura entre duas entidades. Sua aplicação abrange diversos serviços, incluindo HTTPS, SMTPS, FTPS, entre outros. Notavelmente, o TLS desempenha um papel crucial no contexto do HTTPS, assegurando uma comunicação segura entre um cliente e um servidor web. Seu propósito fundamental reside em garantir a confidencialidade e integridade dos dados transmitidos entre ambas as partes, fortalecendo a segurança da troca de informações.

## Resolução

A partir do Wireshark começamos por filtrar a conexão tem relevância para a resolução do CTF.
Para filtrar de acordo com o random dado utilizamos o seguinte codigo : 

```
tls.handshake.random == 52362c11ff0ea3a000e1b48dc2d99e04c6d06ea1a061d5b8ddbf87b001745a27
```

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187816822280560700/image.png?ex=6598439f&is=6585ce9f&hm=cad2a7cb7cbca2048c4ebee4c6cfcd69b6d08f4a43feda321f882cbc90073707&)

Através do log obtido, fomos capazes de analisar que o handshake era iniciado no frame 814. 
Tendo em conta que o protocolo TLS termina quando o servidor envia um New Session Ticket para o cliente referindo que este está pronto para receber os dados, o frame que termina o procedimento é o 819.

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187817805790322688/tls.png?ex=65984489&is=6585cf89&hm=45a71080cc37177b73ca5e2716d6bc4a500ed9430adb6916384d437ebad3a583&)

O cipher suite é o nome correspondente que se encontra na frame 814. Podemos verificar, a partir da seguinte imagem, que o valor é TLS_RSA_WITH_AES_128_CBC_SHA256:

![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187818460978352238/image.png?ex=65984525&is=6585d025&hm=2bd75e24fd55826d1989c5ca3dfc32faa08a2171ff434ffda31fca8ef98de801&)

Para finalizar precisamos de saber qual o tamanho total de dados cifrados trocados entre os dois frames. Para isso é necessário verificar dois frames Application Data que precedem o handshake do frame 814 e consequentemente somar os seus valores (Valores que se encontram no campo Length) : 

Frame 820 : 
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187819217907634176/image.png?ex=659845da&is=6585d0da&hm=d58f286b4bc3f8c1228d57ee48dd24b0a35c3510bc519ad70f4d0bbf250b29db&)

Frame 821: 
![img](https://cdn.discordapp.com/attachments/1153998326274994216/1187819279404511382/image.png?ex=659845e8&is=6585d0e8&hm=fd5f2c0440b2d0c941545188056af5db6ab8d5100c6305a421710480f99c7db4&)


Somando os valores obtêm-se o valor 1264.

Por ultimo, só falta descobrir o tamanho da mensagem crifrada no handshake que se encontra no frame 818. Verificando este valor podemos concluir que o campo Length tem o valor 80.

Em suma, a nossa flag será composta da seguinte forma : 

flag{814-819-TLS_RSA_WITH_AES_128_CBC_SHA256-1264-80}



