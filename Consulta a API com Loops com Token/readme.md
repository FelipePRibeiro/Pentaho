Esse projeto foi realizado após o desafio que coloquei no repositório [aqui](https://github.com/FelipePRibeiro/Pentaho/tree/main/Consulta%20a%20API%20com%20Loops). Foi criado uma nova API, e resolvi fazer também no Pentaho como desafio. A diferença agora é que nesse projeto preciso me conectar a uma API que tem um Token de Autenticação, e também não sei quantas iterações serão feitas, apenas que ao término de puxar todos os dados da API, ela irá me retornar uma messagem de "Dados não encontrados", então a estrutura do Loop será um Do..While. Outra diferença era que antes eu tinha um Offset que era passado no Body de requisição, e agora o Offset se encontra na URL da API.

O projeto ficou assim:

![image](https://user-images.githubusercontent.com/65839541/202053338-75a5ec6a-8280-465d-ba31-64a6c401ccd1.png)

Aqui é o Job orquestrador. Eu inicializo as variáveis principais, checo se o arquivo do resultado da extração completa da API já existe, porque se ele existe eu quero que apague e extraia novamente (Fiz apenas para o conceito do estudo, pois precisei rodar várias vezes para funcionar corretamente rs). E então temos a parte Principal, onde tenho uma transformação que irá pegar o Token e depois acessar a API passando o Token obtido como parâmetro, e ir fazendo a validação se o request foi sucedido e se o retorno é igual a "Dados não encontrados", caso contrário continuo puxando, aumentando o Offset.

![image](https://user-images.githubusercontent.com/65839541/202054717-38b84fbf-f9a0-4671-83a4-1fb860526290.png)

Aqui tenho as inicializações das principais variáveis dentro do Job orquestrador (omiti as informações confidenciais). Temos o vRowCount contendo o tamanho dos pacotes, variável i para controle do Loop, vRowSkip para controle do Offset, e outras variáveis de acesso para a API. Um ponto para chamar a atenção é que deixei visível como fica parcialmente a URL contendo o valor do tamanho do pacote e o Offset, estas contendo as próprias variáveis declaradas acima.

![image](https://user-images.githubusercontent.com/65839541/202055118-02d42973-1f9b-4015-8d12-a3a0fc576d9a.png)

Aqui faço acesso a API para buscar o Token de Autenticação, pegando as variables que foram inicializadas no Job, fazendo a chamada via API, capturando o resultado com o Json Input e setando uma variável contendo o valor do Token.

![image](https://user-images.githubusercontent.com/65839541/202055247-e4e927a6-82c5-419e-808e-225ad80caaad.png)

Agora vem o Step principal, da chamada da API contendo os dados. Pego as variáveis globais do Job e também o Token obtido no step anterior, utilizo o Step Calculator para criar o Bearer Token, faço a chamada da API, e dessa chamada obtenho 3 resultados. O primeiro é realmente os dados em si. O segundo obtém a mensagem do resultado da API, pois ela define quando terminou todos os dados ou não, sempre retornando "Dados não encontrados" quando finalizar. E por último o Status HTTP da requisição, para garantir que o Loop só passe para o próximo só passe se for igual a 200 (Sucedido).

Depois desse passo, O Job orquestrador faz a verificação: o Status HTTP é igual a 200? Se sim, seguir para o próximo Step, caso contrário espere 10s e tente novamente. Seguindo com sucesso, é feito outra validação: a mensagem de retorno da API é igual a "Dados não encontrados"? Se sim vai para um Dummy, pois finalizou, caso contrário faça o controle do Loop, e continue fazendo a requisição, passando para o próximo Offset.

![image](https://user-images.githubusercontent.com/65839541/202055861-f835ccdd-98b3-49d1-9c30-e82e832e9404.png)

![image](https://user-images.githubusercontent.com/65839541/202055952-50f86f48-bd18-4ab7-8a13-86d8ad339af3.png)

O controle do Loop é simples, apenas fazendo um incremento da variável i e do Offset.

O projeto então se resume a fazer um Do While em uma API com Token de autenticação, garantindo que puxe todos os dados necessários.

