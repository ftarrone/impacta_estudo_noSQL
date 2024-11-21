## 1. Fundamentos do MongoDB

### 1.1 Introdução 
Nosso primeiro passo será nos familiarizar com os componentes principais do MongoDB, ajudando você a entender seu propósito e como interagir com eles.

Ao longo deste laboratório, você interagirá com uma instalação _standalone_ do MongoDB. Em ambientes de produção, você provavelmente ira intergir com um _deploy_ mais completo.

### 1.2 Componentes  
Os componentes principais do MongoDB são:

- [**mongosh**]("https://www.mongodb.com/pt-br/docs/mongodb-shell/"): O próprio shell do MongoDB, uma interface interativa em JavaScript e Node.js que você pode usar para se conectar e executar comandos.
- **mongod**: O processo principal (daemon), o processo central do sistema MongoDB.
- **mongos**: O processo que direciona consultas e operações de escrita em um cluster fragmentado.
  
Há também as **Ferramentas de Banco de Dados do MongoDB**, um conjunto de ferramentas para tarefas administrativas, como dump/restore e import/export (dump/restauração e importação/exportação).

### 1.3 Interação dos Componentes
Você precisa de uma ferramenta para acessar uma instância do **mongod**. Você pode usar o shell do MongoDB ou uma ferramenta de terceiros.

O fluxo de interação é o seguinte:

**mongo → conecta-se a → mongod**

Como o **mongod** é o processo do banco de dados, você pode verificar seu status executando o seguinte comando:

`systemctl status mongod`

### 1.4 Topologia

Uma pergunta que você pode ter neste momento é:

> O que posso fazer com esses componentes?

O MongoDB oferece uma implementação diversificada para replicação e disponibilidade de dados, e esses são os componentes necessários para utilizar esses recursos.

As implementações possíveis incluem o seguinte:

**_Standalone_ (Instância Independente):** A instalação básica, um único nó mongod.

**_Replica set_ (Conjunto de Réplicas):** Replicação padrão, várias cópias dos dados em servidores de banco de dados diferentes, oferecendo um nível de tolerância a falhas contra a perda de um único servidor de banco de dados.

![](./resources/images/replica-set-read-write-operations-primary.png)

O _primário_ e os _secundários_ são processos `mongod` que rodam em diferentes locais. Você pode usar o shell do MongoDB (`mongo`) para interagir com eles.

> A configuração mínima recomendada para um conjunto de réplicas é de três membros — um primário e dois secundários.

**_Sharded cluster_ (Cluster Fragmentado):** Esse recurso de cluster do MongoDB permite distribuir dados em várias máquinas (escala horizontal), para conjuntos de dados muito grandes e operações de alto desempenho.

![](./resources/images/sharded-cluster-production-architecture.png)

Em vez de se conectar ao `mongod`, a aplicação deve se conectar a um roteador (`mongos`).

Para tarefas administrativas, você pode usar o shell do MongoDB para se conectar tanto ao `mongos` quanto ao `mongod`.

### 1.5 Construíndo a imagem docker do MongoDB 
1. Para construir a imagem, navegue para a pasta **impacta-labs/mongodb** e execute o seguinte comando:
```bash
docker build -t impacta_labs_mongodb .
```

2. Para iniciar o container, execute o seguinte comando:
```bash
docker run -d -p 27017:27017 --name mongodb-standalone impacta_labs_mongodb
```

#### Shell do MongoDB

Primeiro, vamos tentar conectar usando o shell do MongoDB (**mongosh**) sem nome de usuário e senha.

1. Para acessar o shell do MongoDB, execute o seguinte comando:   
```bash  
docker exec -it mongodb-standalone mongosh
```

2. Para navegar no banco de dados, execute o seguinte comando:
```bash
db
```

3. Para exibir as coleções do banco de dados, execute o seguinte comando:
```bash
show collections
```

4. Para sair do shell do MongoDB, execute o seguinte comando:
```
exit
```

> :warning: Antes de seguir em frente, é importante destacar que essa instância não tem controle de acesso habilitado, o que permite que você se conecte livremente sem nome de usuário e senha. Do ponto de vista da segurança, essa não é uma configuração recomendada para ambientes de produção.

Ótimo! Você usou o shell do MongoDB para se conectar a uma instância MongoDB.

Uma vez conectado, você pode navegar e exibir as estruturas do banco de dados.

### 1.6 Inserindo dados no MongoDB

1. As coleções são objetos capazes de realizar várias operações diferentes. O método insert adiciona documentos a uma coleção. Por exemplo, o seguinte código adiciona um único documento que descreve um livro de Kurt Vonnegut à coleção de livros:

```json
db.books.insertOne( { title: "Mother Night", author: "Kurt Vonnegut, Jr." } )
```

> :memo: É uma boa prática incluir um identificador único em cada documento quando ele é inserido.

2. Em vez de simplesmente adicionar um documento de livro com o título e o nome do autor, uma opção melhor seria incluir um identificador único, como em:

```json
db.books.insertOne( {book_id: 1298747, "title":"Mother Night", "author": "Kurt Vonnegut, Jr."} )
```

> :memo: Diferentes bancos de dados de documentos têm recomendações diferentes para identificadores únicos. O MongoDB adiciona um identificador único se nenhum for fornecido.

3. Em muitos casos, é mais eficiente realizar inserções em massa em vez de uma série de inserções individuais. Por exemplo:

```json
db.books.insertMany([
   {
      "book_id": 1298747,
      "title":"Mother Night",
      "author": "Kurt Vonnegut, Jr."
   },
   {
      "book_id": 639397,
      "title":"Science and the Modern World",
      "author": "Alfred North Whitehead"
   },
   {
      "book_id": 1456701,
      "title":"Foundation and Empire",
      "author": "Isaac Asimov"
   }
])
```

> :memo: Consulte a documentação para verificar os limites no tamanho das inserções em massa. Se você tiver muitos documentos grandes, pode ser necessário realizar várias inserções em massa para garantir que seu conjunto de documentos não exceda o limite de tamanho permitido para inserções em massa.

### 1.7 Deletando dados no MongoDB

1. Para deletar um documento, execute o seguinte comando:
```json
db.books.deleteOne( { "book_id": 1298747 } )
```
2. Para deletar um documento utilizando um atributo específico, execute o seguinte comando:
```json
db.books.deleteOne( { "author": "Kurt Vonnegut, Jr." } )
```
3. Para deletar todos os documentos, execute o seguinte comando:
```json
db.books.deleteMany( {} )
```

> :memo: É importante ter um cuidado especial ao excluir documentos que possam estar referenciados em outros documentos. Bancos de dados relacionais foram projetados para evitar esse tipo de problema, mas bancos de dados de documentos dependem do código da aplicação para gerenciar esse tipo de integridade de dados.

### 1.8 Atualizando dados no MongoDB

1. Para atualizar um documento, execute o seguinte comando:
```json
db.books.updateOne( { "book_id": 1298747 }, { $set: { "quantity": 10 } } )
```

O comando `update` adiciona uma chave se ela não existir e define o valor conforme indicado. Se a chave já existir, o comando update altera o valor associado a ela.

Bancos de dados de documentos às vezes oferecem outros operadores além dos comandos `set`. Por exemplo, o MongoDB possui um operador de incremento `($inc)`, que é usado para aumentar o valor de uma chave pela quantidade especificada.

2. Para exibir todos os documentos, execute o seguinte comando:

```json
db.books.find()
```

3. O comando a seguir alteraria a quantidade de Mother Night de 10 para 15:

```json
db.books.updateOne( { "title": "Mother Night" }, { $inc: { "quantity": 5 } } )
```

### 1.9 Exibindo dados no MongoDB

O método `find` é usado para recuperar documentos de uma coleção. Como você pode esperar, o método `find` aceita um documento de consulta opcional que especifica quais documentos devem ser retornados. Se você quiser apenas um subconjunto de documentos no banco de dados, deverá especificar critérios de seleção usando um documento.

1. Por exemplo, o seguinte comando retorna todos os livros de Kurt Vonnegut, Jr.:

```json
db.books.find( { "author": "Kurt Vonnegut, Jr." } )
```

Consultas mais complexas são construídas usando condicionais e operadores booleanos. 

2. Para recuperar todos os livros com uma quantidade maior ou igual a 10 e menor que 50, você poderia usar o seguinte comando:

```json
db.books.find( { "quantity": { $gte: 10, $lt: 50 } } )
```

As condicionais e operadores booleanos podem ser combinados para criar expressões mais complexas, o MongoDB permite as seguintes expressões:

- `$gt`: maior que
- `$lte`: menor ou igual que
- `$eq`: igual a
- `$ne`: diferente de
- `$in`: contém um valor especificado

Entre outros, veja a [documentação oficial](https://docs.mongodb.com/manual/reference/operator/query-comparison/).

### 1.10 Utilizando agregações no MongoDB

Operações de agregação processam diversos documentos e geram resultados calculados. Você pode usar operações de agregação para:
- Agrupar valores de diversos documentos.
- Executar operações nos dados agrupados para gerar um único resultado.
- Analisar alterações de dados ao longo do tempo.

1. Para agrupar os livros por autor, execute o seguinte comando:
```json
db.books.aggregate([
  { $group: { _id: "$author", total: { $sum: 1 } } }
])
```

2. Para calcular a quantidade total de livros por autor, execute o seguinte comando:
```json
db.books.aggregate([
  { $group: { _id: "$author", total: { $sum: "$quantity" } } }
])
```

## Agora é sua vez

> Para os próximos exercícios, uma nova coleção chamada `filmes` deverá conter documentos com os seguintes campos:
> - `title`
> - `rating`
> - `release_date`

1. Usando a coleção `db.filmes`, escreva um comando para inserir um filme na colecao.
2. Usando a coleção `db.filmes`, escreva um comando para inserir 5 filmes na colecao,
- Um com `rating` 2
- Outro com `rating` 4
- Outro com `rating` 5
- Outro com `rating` 9
- Por fim, outro com `rating` 10.
3. Usando a coleção `db.filmes`, escreva um comando que retorne a média dos filmes.
4. Usando a coleção `db.filmes`, escreva um comando para remover todos os filmes com um `rating` menor ou igual a 5.
5. Usando a coleção `db.filmes`, escreva um comando para exibir todos os filmes com um `rating` maior ou igual a 9.