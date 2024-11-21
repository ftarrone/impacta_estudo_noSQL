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
docker docker build -t impacta_estudo_mongodb .
```

2. Para iniciar o container, execute o seguinte comando:
```bash
docker run -d -p 27017:27017 --name mongodb-standalone impacta_estudo_mongodb
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


> [!IMPORTANT] Antes de seguir em frente, é importante destacar que essa instância não tem controle de acesso habilitado, o que permite que você se conecte livremente sem nome de usuário e senha. Do ponto de vista da segurança, essa não é uma configuração recomendada para ambientes de produção.


> :warning: Antes de seguir em frente, é importante destacar que essa instância não tem controle de acesso habilitado, o que permite que você se conecte livremente sem nome de usuário e senha. Do ponto de vista da segurança, essa não é uma configuração recomendada para ambientes de produção.

Ótimo! Você usou o shell do MongoDB para se conectar a uma instância MongoDB.

Uma vez conectado, você pode navegar e exibir as estruturas do banco de dados.

### 1.6 Inserindo dados no MongoDB

1. Os dados para criação das Collections, está especificada no arquivo ```collection_purchase.json```, e ```collection_description_products.json```

> Lembresse de execultar a insert antes das consultas.

> É uma boa prática incluir um identificador único em cada documento quando ele é inserido.



