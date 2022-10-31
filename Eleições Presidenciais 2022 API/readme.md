Com as eleições de 2022, específicamente no 2º Turno para os Presidentes, quis fazer um projeto bem simples no Pentaho para consultar a API do TSE que retornasse o resultado das eleições, que são utilizadas pelos meios de comunicação para fazer o acompanhamento ao vivo da apuração.

A transformação é simples: Um step de Data Grid para gerar um registro vazio para que a conexão com a API funcione, já que é necessário ter algum input de dado para a conexão REST funcionar. Depois o step de conexão com a API do TSE, passando somente a URL de conexão (https://resultados.tse.jus.br/oficial/ele2022/545/dados-simplificados/br/br-c0001-e000545-r.json). Após isso, um JSON input para pegar da estrutura do JSON somente os campos que retornassem os dados sobre os candidatos a presidência e seus respectivos votos. Por fim um Select Values para tirar os campos contendo o Array do Json e o campo Dummy criado no primeiro Step, e renomear as 2 colunas de votos e percentual dos votos para ficar mais legível.

O resultado final ficou assim:

![image](https://user-images.githubusercontent.com/65839541/198911192-27161ed9-df52-49a2-bd1e-50a035f4b183.png)


