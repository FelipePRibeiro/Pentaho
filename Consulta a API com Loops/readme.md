Tive um desafio no trabalho de fazer uma conexão com API, utilizando o Qlik para fazer o consumo dos dados e inserir em um banco. Consegui fazer, porém quis replicar o mesmo conceito dentro do Pentaho para aprofundar meus conhecimentos nela e mostrar as facilidades/dificuldades que a ferramenta trouxe se comparado ao Qlik. Aqui só me preocupei em fazer o conceito de consultar todos os dados da API utilizando Loops, sem inserção em banco, apenas gerando um arquivo CSV para validação.

O desafio é o seguinte: a API não suporta uma carga "full", ou seja, se ela tem 3 milhões de registros disponíveis, não consigo em apenas um request fazer toda a extração pois ela me retorna erro. Então preciso dividir essa carga em vários requests de N registros que eu vou escolher, e fazer as requisições até terminar o volume de dados. Porém é necessário também fazer uma espécie de Try Catch, pois caso a API me retorne erro por algum motivo, preciso continuar o Loop de onde parei e tentar novamente.

A estrutura essencial da API é básicamente: um campo para fornecer qual a tabela a ser consultada, outro para dizer qual a quantidade de registros a serem trazidos a cada request, e mais um para delimitar um OFFSET, ou seja, quantas linhas pular na API, então se eu passar 100 nesse campo, a API vai trazer o registro 101 para frente. Então com esse campo, vamos fazer nosso Loop, onde a cada incremento do Loop vamos aumentando o OFFSET, até completar todas as Requisições.

O projeto ficou assim:

![image](https://user-images.githubusercontent.com/65839541/198912843-621b7e29-0215-4997-956d-f61c1a821b1d.png)

Aqui é o Job orquestrador. Eu inicializo as variáveis, checo se o arquivo do resultado da extração completa da API já existe, porque se ele existe eu quero que apague e extraia novamente (Fiz apenas para o conceito do estudo, pois precisei rodar várias vezes para funcionar corretamente rs). E então temos a parte Principal, onde tenho uma transformação que irá acessar a API, fazer um request de uma linha apenas para pegar a quantidade de Registros da API, dividir pelo tamanho dos blocos que eu defini na etapa inicial setando as variáveis, pegar o resultado, arredondar para cima e então Subtrair 1. Confuso? rsrs

![image](https://user-images.githubusercontent.com/65839541/198912888-28cd447d-00ce-4287-bcee-c72da968e147.png)

Suponhamos que a API possui 3.230.000 de registros e vou requisitar 50.000 registros por vez. Logo, dividindo o total de registros pelas requsições, teremos 64,6 requisições. Mas não existe 0,6 requsições, então arredondo o resultado para cima, ficando 65 requisições. Porém preciso sempre subtrair 1, pois minha primeira requisição é obrigada a começar com 0, então faremos 65 requisições, porém indo de 0 até 64, por isso o -1.

Após esse step, o próximo é o controle do Loop, que vai fazer o incremento da variável i que controla o Loop e também fará o Offset baseado no valor dessa variável de Controle. Então no primeiro Loop, minha variável de controle é 0, e meu OFFSET também vai ser 0, pois 50000 (que é o que eu quero buscar da API sempre) * i = 0. No segundo Loop, i =1, então meu OFFSET agora é 50000, depois 100000, 150000, e assim vai. Aqui eu também faço o controle de somente incremental a variável em +1 somente se o retorno da API for sucedido, senão preciso manter o valor atual para tentar novamente.

![image](https://user-images.githubusercontent.com/65839541/198912992-ce981836-44af-48af-a093-54bc08a4b3e2.png)

Depois tenho realmente a chamada da API, inicializando com um Data Grid contendo o Body de chamada da API, a Url, credenciais de acessos (encodado em Base64, quis utilizar para aprender, mas poderia ter usado as credenciais que o próprio step do HTPP post oferece), passando depois então as variáveis das transformações anteriores, contendo o OFFSET, a quantidade de registros que vou puxar, e qual a tabela, e tendo que fazer um Split Fields, pois o retorno vem em uma única linha separando todos os campos por Pipe. Aqui também pego o resultado da API e guardo na variável para servir no controle de Loop, e no final guardo o resultado em um CSV sempre dando Apppend (Lembra que apago o arquivo no ínicio do Job? Para garantir que não vou misturar execuções que abortei, estavam erradas com execuções corretas hahaha).

No final, faço 2 avaliações: se o o valor da variável de Controle for igual ao valor da variável que contem quantos Loops tenho que fazer. No exemplo que dei acima seria 64 (65-1). Se i = 64, terminamos o loop, senão precisamos continuar iterando. Depois verifico o resultado da API: foi sucedido? Pode continuar. Não? Espera 10s e tenta novamente de onde parou. :)


