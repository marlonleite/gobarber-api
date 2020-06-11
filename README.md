# GoBarber

## Aula 01 - Configurando a estrutura do projeto

```
yarn add express
```

## Aula 02 - Nodemon & Sucrase

Para utilizar o `import/export` podemos utilizar o babel ou outras ferramentas, mas nesse projeto iremos utilizar `sucrase` que é bem rapido e fácil.

`yarn add sucrase nodemon -D`

Pronto agora só alterar para import/export

Entretanto, não podemos mais rodar node src/index.js para executar o projeto.

Pode ser assim: `yarn sucrase-node src/server.js`

Mas eu quero utilizar o nodemon também.

Nodemon detecta atualiação no código e renicializa o servidor.

Preciso criar um arquivo nodemon.json na raiz do projeto, com a seguinte configuração:
```
{
  "execMap": {
    "js": "sucrase-node"
  }
}
```

Lá no package.json crio um script:

````
"scripts": {
    "dev": "nodemon src/server"
  },
````

e agora posso executar o projeto com: `yarn dev`


## Aula 03 - Conceitos de Docker

Docker ajuda a controlar os serviços externos da aplicação: Banco de Dados, Redis, etc.

### Como funciona?

- Criação de ambientes isolados (container)
	- Você baixa uma imagem com ambiente configurado, você não precisa instalar os softwares na sua máquina e alterar o seu sistema operacional. Quando a gente instala o postgres com Docker ele se torna um subsistema, e fica rodando na máquina virtual do Docker, sem interferir o ambiente, isso é ótimo porque podemos replicar o mesmo ambiente de desenvolvimento ou produção em outras máquinas, sem problema de arquitetura ou diferença em SO.
	- Os containers expoe as portas para podemos nos conectar nos containers.
	- Instalando o postgress, sempre usamos a porta :5432, com mongoDB seria na porta :27017, mas trocar portas no Docker é muito simples.

#### Principais conceitos

	- Imagem: São os principais serviços que iremos utilizar, ex: postres, mongodb, redis, etc.
	- Container: é uma instancia de uma imagem, se tivermos uma imagem do mongodb, podemos criar um ou mais containers do mongodb, até mesmo para servir para outras aplicações na máquina
	- Docker Registry (Docker Hub) é onde podemos visualizar e baixar as imagens (ISOs). é basicamente o repositório de imagens, inclusive podemos criar as nossas próprias imagens e hospedar lá.
	- Dockerfile
		- Receita de uma imagem: Define como a imagem da nossa aplicação irá funcionar em um outro computador, em um ambiente do zero.
		- Dockerfile de exemplo para executar a nossa aplicação:
```
# Partimos de uma imagem existente
FROM node:10
# Definimos a pasta e copiamos o arquivos
WORKDIR /usr/app
COPY . ./
# Instalamos as dependências
RUN yarn
# Qual porta queremos expor?
EXPOSE 3333
# Executamos nossa aplicação
CMD yarn start
```

## Aula 04 - Configurando o Docker

### Mão na Massa

Baixar o docker (Mac, Linux, Windows): [https://docs.docker.com/install/](https://docs.docker.com/install/)

Para ver se está instalado
`docker -v` ou `docker -help`

Repositório de imagens do Docker: [https://hub.docker.com/](https://hub.docker.com/)

Instalando o postgress:
[https://hub.docker.com/_/postgres](https://hub.docker.com/_/postgres)

```
docker run --name database -e POSTGRES_PASSWORD=docker -p 5432:5432 -d postgres
```
Redirecionamento de Portas, toda vez que algum serviço for chamado na porta 5432 do servidor, será redirecionado para 5432 do container no docker:
`-p 5432:5432`

Se já estiver com postgres na máqiuna sem ter sido instaldo pelo Docker, e se estiver executando, você pode alterar na aplicação para: `-p 5433:5432`, isto é, quando for chamado o serviço do postgres 5433, vai ser redirecionado para a porta padrão de dentro do Docker: 5432. Muito legal esse desacoplamento.

Quando já se tem uma imagem no Docker:

pasta executar uma imagem:

```
docker run -d 30bf4f039abe
```

Para executar um container:

```
docker  start a46a366365bb
```

Esses caracteres estranhos é o ID da imagem, para ver basta digitar:
```
docker image ls

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
kartoza/postgis           latest              5a242bc9bf9f        4 months ago        990MB
redis                     alpine              c8eda26fcdab        5 months ago        50.9MB
mongo                     latest              a3abd47e8d61        6 months ago        394MB
mongoclient/mongoclient   latest              436b2a2bbe16        6 months ago        1.11GB
adminer                   latest              709d7ce11f75        6 months ago        83.2MB
postgres                  latest              30bf4f039abe        6 months ago        312MB
mongo                     4                   0da05d84b1fe        7 months ago        394MB
```

Vai listar todas as imagens e seus respectivos IDs.


E para conferir se está rodando, só rodar `docker ps`, com isso ele vai listar todos os containers que estão em execução:

```
docker ps

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
6f0b42548e9e        30bf4f039abe        "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        5432/tcp            goofy_hopper
~
```

Agora ver o banco funcionando, pode conectar com linha de comando no terminal ou instalar uma GUI:

Linux, Mac e Windows: [https://electronjs.org/apps/postbird](https://electronjs.org/apps/postbird)
ou
Mac: [https://eggerapps.at/postico/](https://eggerapps.at/postico/)

Só usar os dados da conexão para poder se conectar no postgres.

e criar o banco de dados: `create database gobarber``

Quando reinicia a máquina, o docker para, para subir novamente só seguir os comandos:

`docker ps -a`  para mostrar todos os container mesmo os que não estão em execução.

e depois executar o comando para subir o container:

```
docker start postgres
```

Pode ser o ID ou o nome do container.

Para ver os logs do container:

```
docker logs postgres
```

O mesmo container pode ser usado para outras aplicações, mas tem como fazer um container apenas para a aplicação.

## Aula 05 - Sequelize & MVC

Sequelize é um [ORM](https://pt.wikipedia.org/wiki/Mapeamento_objeto-relacional) (Object-relational mapping), basicamente ele faz o mapeamento dos objetos como entidade no banco de dados.
Os bancos de dados tem um conceito de Entidade, Tabelas, Atributos, e a aplicação tem o conceito de Objetos, Atributos ou propriedades e métodos ou função. O que o ORM faz é mapear o objeto, criando uma tabela e os atributos mapeando para campos do banco de dados.
O Sequelize também ajuda a fazer as consultas do banco de dados, em vez de usar SQL nativo, podemos usar objetos com seus respectivos métodos, e escrevar javascript para fazer operações de CRUD persistência no BD.

As tabelas do banco de dados se transformam em Models (MVC)

no banco de dados temos:

users, products, productsItem

e no no JS teremos Users.js, Products.js, ProductsItem.js.

Diferença entre SQL e SequelizeSQL:

SQL:
```
INSERT INTO users (name, email) VALUES ("Marlon Leite", "marlonleite@gmail.com")
```
```
SELECT * FROM users WHERE email = "marlonleite@gmail.com" LIMIT 1
```

Sequelize:
```
User.create({ name: 'Marlon Leite' , email: 'marlonleite@gmail.com' , })
```
```
User.findOne({ where: { email: 'marlonleite@gmail.com' } })
```

###  No Sequelize temos também as Migrations:

- Controle de versão para base de dados:
	- Cada alteracão na tabela como adição, remoção de campos ou criação de novas tabelas, é nas migrations que criamos a a estrutura. É um controlador de versão mesmo, pode fazer roolback para desfazer alguma coisa, no banco de dados fica um registro de versão de cada migration que é executada.
- Cada arquivo contém instruções para criação, alteração ou remoção de
tabelas ou colunas;
- Mantém a base atualizada entre todos desenvolvedores do time e também
no ambiente de produção;
- Cada arquivo é uma migration e sua ordenação ocorre por data;

Exemplo de Migration:

```
module.exports = {
    up: (queryInterface, Sequelize) != {
        return queryInterface.createTable('users', {
            id: {
                allowNull: false,
                autoIncrement: true,
                primaryKey: true,
                type: Sequelize.INTEGER
            },
            name: {
                allowNull: false,
                type: Sequelize.STRING
            },
            email: {
                allowNull: false,
                unique: true,
                type: Sequelize.STRING
            }
        })
    },
    down: (queryInterface, Sequelize) != {
        return queryInterface.dropTable('users')
    }
}
```
[https://sequelize.org/master/manual/migrations.html](https://sequelize.org/master/manual/migrations.html)

Obs:

- É possível desfazer uma migração se errarmos algo enquanto estivermos
desenvolvendo a feature;
- Depois que a migration foi enviada para outros devs ou para ambiente de
produção ela JAMAIS poderá ser alterada, uma nova deve ser criada;
- Cada migration deve realizar alterações em apenas uma tabela, você pode
criar várias migrations para alterações maiores;

### Temos também os Seeds
• População da base de dados para desenvolvimento:
	- Podemos utilizar ele para gerar dados em tempo de execução do projeto, quando subimos o projeto ele cria dados fake.
• Muito utilizado para popular dados para testes;
• Executável apenas por código;
• Jamais será utilizado em produção;
	- A ideia aqui é usar apenas os dados fake, para testar o fluxo do sistema e também performance de listas, etc, tem várias libs em JS que geram dados fake que pode ser usado nos Seeds.
• Caso sejam dados que precisam ir para produção, a própria migration
pode manipular dados das tabelas;
	- Os dados que vão para produção devem estar nas Migrations e não no Seed.

### Arquitetura MVC

Model, View, Controller é um arquitetura bem antiga e utilizado nos dias de hoje, onde:

M = Model = Código da estrutura do banco de dados utilizando ORM ou não;
V = View = Código HTML, CSS, JS, JSX, código de criação e manipulação das telas do site/app;
C = Controller = Código JS, que contém a lógica do negócio, é o intermédiário entre o Model e a View

```M <-> C <-> V```

A View faz a requisição, o Controller recebe, processa, chama o banco de dados(Model) o banco retorna para o Controller e repassa para a View a resposta, a qual é renderizada para o usuário.

Exemplo de um Controller:
```
class UserController {
 index() { } !/ Listagem de usuários
 show() { } !/ Exibir um único usuário
 store() { } !/ Cadastrar usuário
 update() { } !/ Alterar usuário
```

## Aula 06 - Padrão de código - Eslint, Prettier & EditorConfig

Padrão de código é muito util quando se está trabalhando com um time, pq cada um pode fazer as coisas de sua maneira e a base de código não vai ficar muito boa, tem desenvolvedor que vai usar var, outro const, e outro let, vai pular linha no final, outro não vai, vai usars export default, e outro não, e isso gera uma bagunça, e ai com isso temos algumas ferramentas que ajudam a definir as regras(padrão) de código e um estilizador de código utilizando as regras, e o mais adotado na comunidade JS, é o Eslint para definir as regras e o Prettier para formatra o código conforme as regras definidas no Eslint.

Mas o ESLINT tbm não tem qualquer regra, você pode usar algumas mais utilizadas no mercado, como as guias de estilos do [Airbnb](https://github.com/airbnb/javascript), [Standard](https://standardjs.com/) e outros, cada um tem suas características e estilos de escrita, ai vai do gosto de cada um, e do padrão que seu framework adota também, no caso Adonis utilizar o Stardard, então é sugestivo você usar o Standard para criar seu projeto seguindo esse estilo.

### Configurando o projeto

Adicionando o eslint como dependencias de desenvolvimento:

```
 yarn add eslint -D
 ```

Feito isso só inicializar o eslint:

``` yarn eslint --init```
Ele vai fazer algumas perguntas e você pode configurar a seu gosto, eu usei a terceira opção:

```
gobarber on  aula6 [!] took 10s
❯ yarn eslint --init
yarn run v1.12.0
$ /Users/tgmarinho/Developer/bootcamp_rocketseat_studies/gobarber/node_modules/.bin/eslint --init
? How would you like to use ESLint?
  To check syntax only
  To check syntax and find problems
❯ To check syntax, find problems, and enforce code style
```
E só seguir respondendo o Eslint. No final ele pede para instalar as dependecias, só instalar e remover o package-lock.json e executar um yarn para atualizar as dependencias, isso eu faço pq não estou usando o npm e sim o yarn como gerenciador de dependência e o eslint em baixo dos panos usa o npm para instalar.

No final ele cria um arquivo: `.eslintrc.js` com as seguintes configurações padrão:
```
module.exports = {
    env: {
        es6: true,
        node: true,
    },
    extends: [
        'airbnb-base',
    ],
    globals: {
        Atomics: 'readonly',
        SharedArrayBuffer: 'readonly',
    },
    parserOptions: {
        ecmaVersion: 2018,
        sourceType: 'module',
    },
    rules: {},
};
```

OBS: Preciso ter o eslint nas extensões do VSCode.

E para o eslint corrigir automaticamente quando salva o arquivo, precisa ter nas settings.json do VSCode a seguinte configuração:

```
  "eslint.autoFixOnSave": true,

  "eslint.validate": [
    "javascript",
    "javascriptreact",
    {
      "language": "typescript",
      "autoFix": true
    },
    {
      "language": "typescriptreact",
      "autoFix": true
    }
  ],
```

Pronto, agora já deve estar ok. Se o VSCode não estiver acusando erro de Eslint no arquivo app.js, pois com padrão Airbnb as aspas tem que ser simples, e deve ter ; no final de cada comando. Então fecha o vscode e abre novamente, ou tenta remover a node_modules e instalar novamente: `rm -rf node_modules/ yarn.lock && yarn`.

No .eslintrc.js, teremos uma definição de novas regras, é tipo como subscrever as regras padrão da guia de estilo airbnb no eslint, isso é necessário algumas vezes devido algum framework que iremos utilizar no dia a dia.
`.eslintrc.js`:
```
 rules: {
    "class-methods-use-this": "off",
    "no-param-reassign": "off",
    camelcase: "off",
    "no-unused-vars": ["error", { argsIgnorePattern: "next" }]
  }
```

### Instalando o Prettier
O Prettier melhora o código, deixando mais bonito, ele faz uma estilização a mais no código, além do que o eslint já faz.

```
 yarn add prettier eslint-config-prettier eslint-plugin-prettier -D
```

e no .eslintrc.js preciso declarar:

```
extends: ["airbnb-base", "prettier"],
plugins: ["prettier"],
```

```
 rules: {
    "prettier/prettier": "error",
    "class-methods-use-this": "off",
    "no-param-reassign": "off",
    camelcase: "off",
    "no-unused-vars": ['error', { argsIgnorePattern:  'next' }],
  }
```

Com isso o prettier está pronto para ser usado, porém, tem algumas regras de conflito entre o prettier e airbnb, então precisa de mais configuração, para desabitar as configurações que o prettier sobrescreveu do airbnb.

Criar o arquivo: `prettier.rc`:
```
{
	"singleQuote":  true,
	"trailingComma":  "es5"
}
```
Defini a regra para manter aspas simples e deixar ; no final de cada instrução de código. Done!

Para corrigir todos os arquivos é só rodar:

```
yarn eslint --fix src --ext .js
```
Legenda: --fix conserta tudo que está na pasta src que tenha a extensão(--ext) de arquivos .js.


Podemos colocar ela no package.json:

```
 "scripts": {
    "dev": "nodemon src/server",
    "eslintify": "yarn eslint --fix src --ext .js"
  },
```

e rodar : `yarn eslintify`

Com isso, agora temos como manter o padrão de código na base de código da aplicação, se receber algum warn ou error, só ajustar conforme a sugestão do ESlint.

Mas e se os outros desenvolvedores não usam a IDE VSCode? Usam o Sublime, Vim, Atom ou WebStorm?

Ai entra o **EditorConfig**

Ele serve para que as regras definidas no Eslint e Prettier sejam aplicadas para todos as IDEs.

para isso basta criar um arquivo: `.editorConfig` na raiz do projeto com as configurações:

```
root = true

[*]
indent_style = space
indent_size = 2
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```
## Aula 07 - Configurando o Sequelize

- Criar estrutura de pastas, dentro da `src`
- adicionar a dependência: `yarn add sequelize` no projeto
- adicionar a interface de linha de comando do sequelize: `yarn add sequelize-cli -D`
- criar o arquivo `.sequelizerc` na raiz do projeto para poder configurar os caminhos para as pastas de models, config, para rodar os comandos sequelize-cli:
```
const { resolve } = require('path');

modules.exports = {
  config: resolve(__dirname, 'src', 'config', 'database.js'),
  'models-path': resolve(__dirname, 'src', 'app', 'models'),
  'migrations-path': resolve(__dirname, 'src', 'database', 'migrations'),
  'seeders-path': resolve(__dirname, 'src', 'database', 'seeds'),
};
```

Configurando o database:
- adiciono as dependencias: ```yarn add pg pg-hstore```
- no arquivo config/database.js:
```
module.exports = {
  dialect: 'postgres',
  host: 'localhost',
  username: 'postgres',
  password: 'docker',
  database: 'gobarber',
  define: {
    timestamps: true, // garante que será criado um atributo: created_at e updated_at na tabela do banco de dados.
    underscored: true, // permite o ORM criar nome de tabelas como products_item
    underscoredAll: true, // permite o ORM criar nome dos atributos com caixa baixa e _ em vez de camelCase, pois esse é a convenção de escrita no banco de dados
  },
};
```

## Aula 08 -Migração de usuário

Para criar as migrations basta rodar  comando:

```
yarn sequelize migration:create --name=create-users
```

Com isso ele vai criar um arquivo:
```
20190913144153-create-users.js
```
Com um template:
```
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    /*
      Add altering commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.createTable('users', { id: Sequelize.INTEGER });
    */
  },

  down: (queryInterface, Sequelize) => {
    /*
      Add reverting commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.dropTable('users');
    */
  },
};

```

Método up quando a migration é executada e método down para fazer um rollback.

### Migration de Usuário

```
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('users', {
      id: {
        type: Sequelize.INTEGER,
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      email: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true,
      },
      password_hash: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      provider: {
        type: Sequelize.BOOLEAN,
        defaultValue: false,
        allowNull: false,
      },
      created_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
      updated_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
    });
  },

  down: queryInterface => {
    return queryInterface.dropTable('users');
  },
};

```


Para rodar a migration:

```
❯ yarn sequelize db:migrate
yarn run v1.12.0
$ /Users/tgmarinho/Developer/bootcamp_rocketseat_studies/gobarber/node_modules/.bin/sequelize db:migrate

Sequelize CLI [Node: 10.16.3, CLI: 5.5.1, ORM: 5.18.4]

Loaded configuration file "src/config/database.js".
== 20190913144153-create-users: migrating =======
== 20190913144153-create-users: migrated (0.040s)

✨  Done in 1.02s.
```

E ai podemos ver o DDL lá na GUI do Postgres:

```

CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name character varying(255) NOT NULL,
    email character varying(255) NOT NULL UNIQUE,
    password_hash character varying(255) NOT NULL,
    provider boolean NOT NULL DEFAULT false,
    created_at timestamp with time zone NOT NULL,
    updated_at timestamp with time zone NOT NULL
);

-- Indices -------------------------------------------------------

CREATE UNIQUE INDEX users_pkey ON users(id int4_ops);
CREATE UNIQUE INDEX users_email_key ON users(email text_ops);

```

Alêm da tabela users, é criada uma tabela SequelizeMeta que tem os registros de cada migration que foram executadas.

Para desfazer as migrations:

```
yarn sequelize db:migrate:undo
```

Com isso a tabela users não existirá mais.

Desfazer tudo, com isso desfazer todas as migrations que foram executadas e não apenas a última.

```
yarn sequelize db:migrate:undoAll
```

## Aula 09 - Criação do Model de Usuário

Criar um model o sequelize, dentro da pasta models, criar uma arquivo: user.js:

```
import Sequelize, { Model } from 'sequelize';

class User extends Model {
  static init(sequelize) {
    super.init(
      {
        name: Sequelize.STRING,
        email: Sequelize.STRING,
        password_hash: Sequelize.STRING,
        provider: Sequelize.BOOLEAN,
      },
      {
        sequelize,
      }
    );
  }
}

export default User;

```

Criei a classe User que extend de Model do sequelize para receber todos métodos que tem na Model, agora a nossa classe de domínio User é uma Model, devido a herança.
De dentro do método statico init passo uma parametro sequelize do Model, e chamo o super da model, informando os atributos da tabela users, e com isso é feito o mapeamento do ORM entre as entidades do banco de dados e o objeto da aplicação, para entidade User.

## Aula 10 - Loader de Models

Para conectar a aplicação com banco de dados  e carregar os models, temos que criar um o arquivo index.js na pasta database.
```
import Sequelize from 'sequelize';
import User from '../app/models/User';
import databaseConfig from '../config/database';

const models = [User];

class Database {
  constructor() {
    this.init();
  }

  init() {
    this.connection = new Sequelize(databaseConfig);
    models.map(model => model.init(this.connection));
  }
}
export default new Database();
```

Quando esse arquivo é importando, ele recebe uma instância do Database, que chama a função init, que instancia para o this.connection a Sequelize com as configurações de conexão com banco de dados. E para cada model que eu importei eu passo a conexão.

E agora só testar, veja o código da aula.

```
// Quando chamo a rota '/', cadastro o usuário e retorno os dados do banco de dados
// http://localhost:3333/

{
  "id": 1,
  "name": "Marlon Leite",
  "email": "marlonleite@gmail.com",
  "password_hash": "123456",
  "updatedAt": "2019-09-13T15:39:29.116Z",
  "createdAt": "2019-09-13T15:39:29.116Z",
  "provider": false
}
```

## Aula 11 - Criando usuário

* Criar controller de Usuário
* Criar a rota de users para receber a requisição e passar o UserController.store para que seja executado quando a rota for chamada.
* Validar se já existe o usuário, pois o email é um campo único na base de dados (validação no backend é importante)

## Aula 12 - Gerando hash para a senha

Quando o usuário digita a senha e envia para o controllers, queremos que seja gerado um hash para salvar a senha no banco de dados, e posteriomente quando ele for fazer login, ele digita a senha normal, e geramos um hash e comparamos com o hash que foi salvo no password_hash do banco de dados, se for igual, ok, está autenticado.

Para fazer isso precisamos de uma lib para gerar o hash do password:
```
yarn add bcryptjs
```

Bcryptjs é utilizado no model de User, criamos um campo virtual, que é utilizado para receber o password do frontend e que é hashead para através da lib bcrypt para a variável password_hash que essa sim é uma String que é persistida no banco de dados.

## Aula 13 - Conceitos de JWT

Json Web Token([JWT](https://jwt.io/)) , server para fazer autenticação.

**JWT (JSON Web Token)** é um sistema de transferência de dados que pode ser enviado via POST ou em um cabeçalho HTTP (header) de maneira “segura”, essa informação é assinada digitalmente por um algoritmo HMAC, ou um par de chaves pública/privada usando RSA. ([Saiba mais...](https://imasters.com.br/desenvolvimento/json-web-token-conhecendo-o-jwt-na-teoria-e-na-pratica))

Gera um token com headers (tipo de token, algoritmo), Payload (Dados adicionais) e Assinatura ( o que garante a veracidade do token, não pode ser modificado)

## Aula 14 - Autenticação JWT

Vamos usar a lib [jsonwebtoken](https://github.com/auth0/node-jsonwebtoken):

 ```yarn add jsonwebtoken```

Para criar a autenticação do usuário, podemos criar um controller: SessionController.js que server para tratar a autenticação e não a criação de usuário.

Para gerar string aleatória (secret).
[https://www.md5online.org/](https://www.md5online.org/)


## Aula 15 - Middleware de autenticação

Criar um Middleware para **bloquear** o usuário se ele não estiver logado na aplicação.

Para garantir que o usuário está logado, ele tem que ter o token no header.

Então quando for chamar a rota de update usando o método `put` na rota `/users`, antes de chamar o `UserController.update`, tem que chamar o `authMiddleware`, que vai verificar se o `token` está presente na requisição.

Código do Middleware:
`app/middleware/auth.js`:

```
export default (req, res, next) => {
  const authHeader = req.headers.authorization;
  console.log(authHeader);

  next();
};
```

`routes.js`:

```
routes.put('/users', authMiddleware, UserController.update);
```

Ou podemos utilizar de forma global, e toda a rota abaixo de use(authMiddleware) devem ser autenticadas:

```
import { Router } from 'express';
import UserController from './app/controllers/UserController';
import SessionController from './app/controllers/SessionController';
import authMiddleware from './app/middlewares/auth';

const routes = new Router();

routes.post('/users', UserController.store);
routes.post('/sessions', SessionController.store);

// Todas as rotas que forem chamadas a partir daqui tem que ser autenticada
routes.use(authMiddleware);
routes.put('/users', UserController.update);

export default routes;
```

Middleware de autenticação completo:

```
import jwt from 'jsonwebtoken';
import { promisify } from 'util';
import authConfig from '../../config/auth';

export default async (req, res, next) => {
  const authHeader = req.headers.authorization;

  if (!authHeader) {
    return res.status(401).json({ error: 'Token not provided' });
  }

  const [, token] = authHeader.split(' ');

  try {
    const decoded = await promisify(jwt.verify)(token, authConfig.secret);

    req.userId = decoded.id;
    return next();
  } catch (error) {
    return res.status(401).json({ error: 'Token invalid' });
  }
};
```

Estou utilizando promisfy do node para transformar o callback do jwt.verify em promise e poder utilizar o async/await para poder resolver a promisse, fica muito melhor. Invocando o promisy, passo a função que contém o callback, ele me retorna uma verify em promise e passos os valores da função. E assim verifico se o token é valido.

O decode recebe o ID do usuário por foi o atributo que foi passado como payload na geração do token (ver arquivo `SessionController.js`):

```
token: jwt.sign({ id }, authConf.secret, {
	expiresIn: authConf.expireIn,
}),
```

Além do id, o decode recebe o exp: que é a data e hora em timestamp que o token irá expirar.

Foi feito uma trick do JS para remover o Baerer da string do token, o token vem como uma String:

```
Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NiwiaWF0IjoxNTY4NDA1MDAyLCJleHAiOjE1NjkwMDk4MDJ9.NPwa4vr80wAeEJvX9XWNMQAsUWXaDoSUwuw1KAR4wVw
```

E precisamos apenas do valor do token, para isso desestruturamos o array, que foi gerado pelo split(' '), que cortou a string em dois, pelos espaços (q no caso só tem um espaço na string) e retornou o array:

```
const [, token] = authHeader.split(' ');
```

```
['Bearer','eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6NiwiaWF0IjoxNTY4NDA1MDAyLCJleHAiOjE1NjkwMDk4MDJ9.NPwa4vr80wAeEJvX9XWNMQAsUWXaDoSUwuw1KAR4wVw']
```

E como não precisamos do `Baerer`, então fazemos apenas: `const [, token]`, menosprezamos o valor da primeira posição, pois a palavra `Baerer` é insignificante no nosso contexto e ficamos apenas com o `token`.

E por fim pegamos o ID do usuário que estava no payload do token e guardamos na requisição:

```
req.userId = decoded.id;
```

E agora com isso no método update do UserController.js teremos acesso, e em todos os controllers que passarem pela verificação de autenticação:

`UserController.js`:

```
 async update(req, res) {
    console.log(req.userId);
    return res.json({ ok: true });
}
```

## Aula 16 - Update do usuário

Código do update do usuário no arquivo: `UserController.js`

```
  async update(req, res) {
    const { email, oldPassword } = req.body;
    const user = await User.findByPk(req.userId);
    if (email !== user.email) {
      const userExists = await User.findOne({
        where: { email },
      });
      if (userExists) {
        return res.status(400).json({ error: 'User already exists.' });
      }
    }
    // só faço isso se ele informou a senha antiga, isto é, quer alterar a senha
    if (oldPassword && !(await user.checkPassword(oldPassword))) {
      return res.status(401).json({ error: 'Password does not match.' });
    }

    const { id, name, provider } = await user.update(req.body);

    return res.json({ id, name, email, provider });
  }
```

Nesse código:

- Sempre tenho que ter o email do usuário;
- Usuário pode informar a senha antiga, caso queira alterar a senha;
- Pode alterar todos os atributos;
- É feito uma verificação se a senha antiga é a senha atual do usuário
- Aproveito a instância de user que veio do banco de dados no findByPk e altero os dados do usuário, o qual retorna os novos usuário
- Envio via json todos os dados do usuário, exceto a senha.


## Aula 17 - Validando dados de entrada

Vamos validar os dados do usuário, é uma boa pratica ter a validação do usuário no frontend no backend, a vantagem de estar no frontend é que a validação é mais rápida, não precisa ir diretamente no servidor para poder verificar se tem algum dado errado ou faltando, ganha em velocidade, também em menos tráfego ao servidor e principamente na segurança. Ter só a validação no frontend não é uma boa prática, na verdade é uma péssima prática.

Vamos validar o frontend com biblioteca [Yup](https://github.com/jquense/yup), que faz uma validação no schema, Schema Validation:

```
yarn add yup
```

### Trecho de código com Yup Validation

Esse código é do método update no UserController.js

```
import  *  as Yup from  'yup';

  const schema = Yup.object().shape({
      name: Yup.string(),
      email: Yup.string().email(),
      oldPassword: Yup.string().min(6),
      password: Yup.string()
        .min(6)
        .when('oldPassword', (oldPassword, field) =>
          oldPassword ? field.required() : field
        ),
      confirmPassword: Yup.string().when('password', (password, field) =>
        password ? field.required().oneOf([Yup.ref('password')]) : field
      ),
    });
```

Uso o Yup para criar um schema, passado um objeto com o corpo definido por mim, detalhe para campos condicionais, caso o oldPassword é informado o campo password deve ser obrigatório (required), o field no segundo parametro é o password.

Mas se o usuário digitar a senha, ele precisa confirmar a senha, então ele informa o confirmPassword, e ambos precisam ser iguais, então uso a função `oneOf` que recebe um array, e o Yup tem a referências de todos os campos, então uso: `Yup.ref('password')`.

Defino o schema, agora é só validar com os dados que vieram da requisição (req.body):

```
if (!(await schema.isValid(req.body))) {
	return res.status(400).json({ error:  'Validation fails' });
}
```

o método é assincrono então uso await, se tiver algo que não atende os requisitos do Schema Valitation então retorna uma mensagem para o usuário com error: 'Validation Fails'.

Podemos validar os dados informados no Login, dentro do SessionController.js:

```
  const schema = Yup.object().shape({
      email: Yup.string()
        .email()
        .required(),
      password: Yup.string().required(),
    });

    if (!(await schema.isValid(req.body))) {
      return res.status(400).json({ error: 'Validation fails' });
    }
```

Aqui verificamos apenas se o usuário informou o email e senha.


## Aula 18 - Configurando Multer

### Upload de imagem

Usuário seleciona a imagem, o upload já é feito e o servidor retorna o ID da imagem.
E no json no cadastro de usuário por exemplo, envia o ID da imagem.

Utilizando o [multer](https://github.com/expressjs/multer) para upload de arquivos.

Quando precisa enviar imagem para o servidor, tem que ser como `Multpart-data` (Multpart Form) e não `json`.

Instalando o `multer`:

```
yarn add multer
```

Criar uma pasta fora do `src`, para armazenar as imagens: `tmp/uploads`, dentro da pasta `tmp` criar outra pasta `uploads`, onde vai ficar os arquivos físicos de uploads de arquivos.

Criar um arquivo de configuração `multer.js` de dentro da `config`.

```
import multer from 'multer';
import crypto from 'crypto';
import { extname, resolve } from 'path';

export default {
  storage: multer.diskStorage({
	// Local onde o arquivo será salvo na máquina do servidor
    destination: resolve(__dirname, '..', '..', 'tmp', 'uploads'),
    // Gerando o nome da imagem como um hash usando a lib nativa do node: crypto
    filename: (req, file, cb) => {
      crypto.randomBytes(16, (err, res) => {
        if (err) return cb(err);
        return cb(null, res.toString('hex') + extname(file.originalname));
      });
    },
  }),
};
```

Depois criar um rota:

```
import multer from  'multer';
const upload =  multer(multerConfig);

const upload =  multer(multerConfig);

routes.post('/files', upload.single('file'), (req, res) => {
	return res.json({ ok:  true });
});
```

A rota tem que usar o método post, e o corpo da requisição tem que ser um `multpart-form` em vez de `json`.

Depois adicionar um atributo `file` e adicionar o arquivo nesse atributo.

`upload.single('file')` significa que vou enviar apenas um arquivo dentro da propriedade `file`.

Essa lib multer permite envio de multiplos arquivos.

## Aula 19 - Avatar do Usuário

### Salvando informações do arquivo na base de dados

O atríbuto `req`tem disponível a variável `file` do upload de arquivos que armazena algumas informações sobre o arquivo de upload:

```
{
  "fieldname": "file",
  "originalname": "code-hoc.png",
  "encoding": "7bit",
  "mimetype": "image/png",
  "destination": "/Users/xxx/Developer/bootcamp_rocketseat_studies/gobarber/tmp/uploads",
  "filename": "1d05508938b533ef539026149c597ed5.png",
  "path": "/Users/xxx/Developer/bootcamp_rocketseat_studies/gobarber/tmp/uploads/1d05508938b533ef539026149c597ed5.png",
  "size": 115050
}
```

originalname: é o nome original do arquivo que estava na máquina cliente, que fez o upload.
filename: é o novo nome da imagem que vai ficar salva no servidor.

Para lidar melhor com o upload de arquivo, vou criar o `FileController.js` que conterá a lógica para salvar no banco de dados as referências dos arquivos de upload.

Para salvar os dados do arquivo, vamos criar a tabela files no banco de dados, criando o arquivo de migration.

```
yarn sequelize migration:create --name=create-files
```

E terminar de configurar:

```
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.createTable('files', {
      id: {
        type: Sequelize.INTEGER,
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
      },
      name: {
        type: Sequelize.STRING,
        allowNull: false,
      },
      path: {
        type: Sequelize.STRING,
        allowNull: false,
        unique: true,
      },
      created_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
      updated_at: {
        type: Sequelize.DATE,
        allowNull: false,
      },
    });
  },

  down: queryInterface => {
    return queryInterface.dropTable('files');
  },
};
```

E para gerar a tabela files no banco de dados conforme a migration, só executar no terminal:

```
yarn sequelize db:migrate
```

Depois criar o Model File:

```
import Sequelize, { Model } from 'sequelize';

class File extends Model {
  static init(sequelize) {
    super.init(
      {
        name: Sequelize.STRING,
        path: Sequelize.STRING,
      },
      {
        sequelize,
      }
    );

    return this;
  }
}

export default File;
```

E inserir o Model File no arquivo database/index.js

```
...
import File from  '../app/models/File';
...
const models = [User, File];
...
```

E agora atualizar o FileController.js para poder receber os dados da requisição do arquivo de upload, e salvar no banco de dados as referências:

```
import File from '../models/File';

class FileController {
  async store(req, res) {
    const { originalname: name, filename: path } = req.file;

    const file = await File.create({ name, path });

    return res.json(file);
  }
}

export default new FileController();
```

Agora na hora que enviar novamente o arquivo, a tabela dados irá ser preenchida.

### Relacionando o usuário com imagem de avatar (user <-> files)

Para fazer o relacionamento precisamsos adicionar as chaves primária de files no users.

Para isso teremos que criar uma migration para atualizar essas tabelas:

```
yarn sequelize migration:create --name=add-avatar-field-to-users
```

Adicionamos a coluna `avatar_id` de dentro da tabela `users`, sendo referenciadas pela tabela `files` no atributo `ID` que é a chave primária da tabela `files`. E quando desfazer a migration é só apagar o atributo `avatar_id` de `users`.

`onUpdate: 'CASCADE'`: Quando atualiza a imagem, altera no usuário
`onDelete: 'SET NULL'`: Quando deletar o avatar deixa o avatar_id como null

```
module.exports = {
  up: (queryInterface, Sequelize) => {
    return queryInterface.addColumn('users', 'avatar_id', {
      type: Sequelize.INTEGER,
      references: { model: 'files', key: 'id' },
      onUpdate: 'CASCADE',
      onDelete: 'SET NULL',
      allowNull: true,
    });
  },

  down: queryInterface => {
    return queryInterface.removeColumn('users', 'avatar_id');
  },
};
```

Só executar: `yarn sequelize db:migrate` para executar a alteração na tabela `users`.

Depois precisa relacionar o Users com Files de dentro do Model de users no código.

Adicionando um método para associar as duas entidades:

`Users.js`:

```
...
static associate(models) {
	this.belongsTo(models.File, { foreignKey: 'avatar_id' });
}
...
```

E dentro do `database/index.js`, acresento mais um `map`, para poder executar (apenas nas classes que contém o método associate) a associação e passar os models para o associate:

```
 models
      .map(model => model.init(this.connection))
      .map(model => model.associate && model.associate(this.connection.models));
```

Pronto, agora sim, na hora de salvar o usuário a associação irá ocorrer e o ID que for informado no files estará no users.

## Aula 20 - Listagem de prestadores de serviços

Vamos criar a listagem de usuários prestadores de serviço, embora seja a mesma entidade de usuários, iremos criar um controller específico para o tipo de usuário provider.

- Criamos uma nova rota para /providers
- Criamos um novo controller ProviderController.js
- Buscamos todos os usuários que são providers, e trouxemos inclusive o relacionamento com o Avatar, excluindo campos desnecessários
- Criamos um campo virtual url e para montar a URL da imagem
- Demos um apelido para File, para se chamar avatar na criação do objeto de retorno da requisição.
- Permitimos o servidor servir o arquivo de forma estática

## Aula 21 - Migration e model de agendamento

- Criar model e migration da tabela de agendamento
- Todas vez que usuário marcar um agendamento com algum provedor de seriviço, irá gerar um registro na tabela de agendamento
- Relacionar usuário cliente e o usuário provider na tabela de agendamento
- Referenciar o model Appointment no models dentro do database/index.js

## Aula 22 - Agendamento de serviço

- Criar a rota de agendamento e o controller
- O cliente pode selecionar apenas um usuário que seja provider

## Aula 23 - Validações de agendamento

- Validar se a data de agendamento é uma data futura
- Validar se a data de agendamento já está ocupada para o mesmo prestador de serviço
- Permitir agendar de hora em hora
- Utilizar a [DateFNS](https://date-fns.org/)) pra lidar com datas
	- Para instalar: `yarn add date-fns@next`
	- `import { startOfHour, parseISO } from  'date-fns';`
	- `parseISO` transforma `"2019-10-01T18:00:00-04:00"` em Objeto data do JS
	- `startOfHour` despreza os minutos e segundos, e retorna apenas da hora. 18h35 fica apenas 18h.
	- `isBefore(x,y)` verifica se a data do primeiro parametro é anterior a do segundo parametro
- Não permitir agendamento duplicado para o o prestador na mesma hora.

## Aula 24 - Listando agendamentos do usuário

Mostrar todos os agendamentos do usuário logado e mostrar seus prestadores de serviços

- Criar nova rota com método get no `routes.js` para o AppointmentController no método index.
- Buscando todos os agendamentos do usuário logado, que não estão cancelados, tranzendo o usuário provider, prestador de serviço com o seu avatar. Ordenado por data, trazendo apenas os atributos id e data do agendamento.

```
class AppointmentController {
  async index(req, res) {
    const appointments = await Appointment.findAll({
      where: {
        user_id: req.userId,
        canceled_at: null,
      },
      order: ['date'],
      attributes: ['id', 'date'],
      include: [
        {
          model: User,
          as: 'provider',
          attributes: ['id', 'name'],
          include: [
            {
              model: File,
              as: 'avatar',
              attributes: ['id', 'path', 'url'],
            },
          ],
        },
      ],
    });
    return res.json(appointments);
  }
```

## Aula 25 - Aplicando paginação

- No Insomnia utiliza a aba query para passar parametros: page = 1 ou 2, n...
- Pegar o page: `const { page =  1 } = req.query;``
- Definir um limit, de quantos em quantos vou trazer e um offset para definir o corte:
`AppointmentController.js`:
```
...
limit:  20,
offset: (page -  1) *  20,
...
```

## Aula 26 - Listando agenda do prestador

- Mostar no painel do prestador de serviço a listagem de sua agenda
- Criar uma nova rota para agenda do provider (schedule)
- Criar um novo controller: ScheduleController.js
- Verificar se o usuário logado é um provider(prestador)
- buscar agenda pela data e fazer um parseISO
- buscar os agendamentos do provedor logado, que não esteja cancelado e que a data seja no ínicio do dia buscado até o final do dia.

```
import { startOfDay, endOfDay, parseISO } from 'date-fns';
import { Op } from 'sequelize';
import Appointment from '../models/Appointment';
import User from '../models/User';

class ScheduleController {
  async index(req, res) {
    const checkUserProvider = await User.findOne({
      where: {
        id: req.userId,
        provider: true,
      },
    });

    if (!checkUserProvider) {
      return res.status(401).json({ error: 'User is not a provider' });
    }

    const { date } = req.query;
    const parsedDate = parseISO(date);

    const appointments = await Appointment.findAll({
      where: {
        provider_id: req.userId,
        canceled_at: null,
        date: {
          [Op.between]: [startOfDay(parsedDate), endOfDay(parsedDate)],
        },
      },
      order: ['date'],
    });
    return res.json(appointments);
  }
}

export default new ScheduleController();
```

## Aula 27 - Configurando MongoDB

- Conectar a aplicação com banco de dados não estrutural, pois iremos armazenar alguns dados que não são estruturados.

- Criando um container do Mongo utilizando o Docker para baixar e configurar:

```
docker run --name mongobarber -p 27017:27017 -d  -t mongo
```

Para saber se o mongo está funcionando: [http://localhost:27017/](http://localhost:27017/)
Ou executar `docker ps` pra ver os containers em execução.

- Instalar o Mongoose para ser o ORM, semelhante ao Sequelize do SQL:

```
yarn add mongoose
```
- Utilizando o Mongoose

Vamos inicializar o mongo dentro do `database/index.js`, assim como foi iniciando o postgres.

Criamos a função `mongo()` que contém a configuração de conexão com `mongodb`, como não foi criado um usuário e senha na criação do container, então não precisa informar na string de conexão, basta só informa o endereço da máquina (host), e passamos o nome da collection que é criada assim que a conexão é efetuada, ela não precisa exisitir primeiro, ao contrário da conexão com postgres (SQL).
```
import Sequelize from 'sequelize';
import mongoose from 'mongoose';
import User from '../app/models/User';
import File from '../app/models/File';
import Appointment from '../app/models/Appointment';
import databaseConfig from '../config/database';

const models = [User, File, Appointment];

class Database {
  constructor() {
    this.init();
    this.mongo();
  }

  init() {
    this.connection = new Sequelize(databaseConfig);

    models
      .map(model => model.init(this.connection))
      .map(model => model.associate && model.associate(this.connection.models));
  }

  mongo() {
    this.mongoConnection = mongoose.connect(
      'mongodb://localhost:27017/gobarber',
      {
        useNewUrlParser: true, // estou utilizando um formato novo na string de conexão
        useFindAndModify: true, // para poder buscar e atualizar os registros
        useUnifiedTopology: true, // DeprecationWarning apareceu no console então eu estou usando, conforme a recomendação do mongo
      }
    );
  }
}

export default new Database();
```

## Aula 28 - Notificando novos agendamentos

- Vamos enviar uma notificação para o prestador de serviço toda vez que receber um novo agendamento, e por isso vamos utilizar o MongoDB, vamos adicionar as notificações dentro do mongo.

- Criar os schemas do mongo, semelhante ao model das tabelas.

- MongoDB tem Schema Free, um registro na Collection pode ter um campo e outro registro não ter o campo, isso difere das Tabelas, pois se um registro não tem um certo atributo, ele deve ter o campo com o valor `null`, e no mongo, o campo/registro nem precista existir. Por isso é Schema Free e se chama NOSQL, não tem estrutura. E no mongo também não tem as migrations.
- As notificações não tem muita estrutura, ela armazena o ID do usuário, e as notificações não precisam se relacionar, nem precisa haver consultas com essa coleção (entidade).

- Criando o Schema de Notification:

`app/schema/Notification.js`:
```
import mongoose from 'mongoose';

const NotificationSchema = new mongoose.Schema(
  {
    content: {
      type: String,
      required: true,
    },
    user: {
      type: Number,
      required: true,
    },
    read: {
      type: Boolean,
      required: true,
      default: false,
    },
  },
  {
    timestamps: true, // Cria os campos created_at e update_at automagicamente
  }
);

export default mongoose.model('Notification', NotificationSchema);
```

E no controller utilizamos o schema de Notification para criar um registro no mongo:

```
...
import { startOfHour, parseISO, isBefore, format } from  'date-fns';
import pt from  'date-fns/locale/pt';
import Notification from  '../schemas/Notification';
...
...
 /**
     * Notify appointment provider
     */
    const user = await User.findByPk(req.userId);
    const formattedDate = format(
      hourStart,
      "'dia' dd 'de' MMMM', às' H:mm'h'",
      {
        locale: pt,
      }
    );
    await Notification.create({
      content: `Novo agendamento de ${user.name} para o ${formattedDate}`,
      user: provider_id,
    });
...
```

Após criar o agendamento, eu crio uma notificação e armazeno no banco de dados, esse dado não é estruturado, eu lanço diretamente no content o nome do usuário e a data, eu não preciso relacionar Notification com Users e nem Appointments e fazer joins e mais joins de SQL, pois a notificação é do estado atual, que o usuário está, se ele mudar o nome dele, isso não é importante para a notificação nesse momento, e com o mongodb ganhasse bastante performance e facilidade justamente por não ter que escrever Queries SQL gigantes e outra vantagem é que podemos escrever em JS para fazer consultas no mongodb.

## Aula 29 - Listando notificações do usuário

- Criar uma rota notifications
- Criar o controller `NotificationController.js`:

```
import Notification from '../schemas/Notification';
import User from '../models/User';

class NotificationController {
  async index(req, res) {
    /**
     * Check if loggedUser is a provider
     */
    const checkIsProvider = await User.findOne({
      where: {
        id: req.userId,
        provider: true,
      },
    });

    if (!checkIsProvider) {
      return res
        .status(401)
        .json({ error: 'Only provider can load notifications' });
    }

    const notifications = await Notification.find({
      user: req.userId,
    })
      .sort({ createdAt: 'desc' })
      .limit(20);

    return res.json(notifications);
  }
}

export default new NotificationController();
```
## Aula 30 - Marcar notificações como lidas

- Criar nova rota put notifications

Utilizando `findByIdAndUpdate` do mongo para buscar e atualizar os registros, para isso funcionar tem que estar marcado lá na conexão com mongodb: `useFindAndModify:  true`:

```
mongo() {
    this.mongoConnection = mongoose.connect(
      'mongodb://localhost:27017/gobarber',
      {
        useNewUrlParser: true,
        useFindAndModify: false, // Agora posso usar findByIdAndUpdate
        useUnifiedTopology: true,
      }
    );
```
Entendendo a query:

```
 const notification = await Notification.findByIdAndUpdate(
      req.params.id,
      {
        read: true,
      },
      { new: true }
    );
```

- `findByIdAndUpdate` = Busco e atualizo o registro
- `req.params.id` = é o id do registro no mongo que foi passado como parametro na query de consulta no frontend
 -  `{ read: true }`  é o valor que eu quero alterar, sempre por default é falso o valor registrado, e agora quero alterar para true, pois foi lida.
 - `{ new:  true }` retorno para o usuário a notificação para dentro de const  notification atualizado

## Aula 31 - Cencelamento de agendamento

- Usuário só pode cancelar o agendamento se for duas horas antes do evento
- Criar uma rota delete appointment passando o id.
	- `routes.delete('/appointments/:id', AppointmentController.delete);`
- Criar o método delete no AppointmentController.js:
```
...
import { startOfHour, parseISO, isBefore, format } from  'date-fns';
...
...
async delete(req, res) {
    const appointment = await Appointment.findByPk(req.params.id);

    if (appointment.user_id !== req.userId) {
      return res.status(401).json({
        error: "You don't have permission to cancel this appointment.",
      });
    }

    // removo duas horas da data agendada
    const dateWithSub = subHours(appointment.date, 2);
    const NOW = new Date();
    if (isBefore(dateWithSub, NOW)) {
      return res.status(401).json({
        error: 'You can only cancel appointment 2 hours in advance.',
      });
    }

    appointment.canceled_at = NOW;

    await appointment.save();

    return res.json(appointment);
```

- Busco o appointment com o valor passando no query params com id do appointment
- Verifico se o usuário logado é dono do appointment
- Removo duas horas da data agendada, pois só pode cancelar o evento duas horas antes
- verifico se a hora appointment é antes da hora atual, se for envio mensagem de error
- Se não for continua o fluxo setando no canceled_at a data atual.
- Salvo o appointment e retorno para o usuário


## Aula 32 - Configurando Nodemailer

- Vamos enviar um email para o prestador de serviço quando um agendamento for cancelado.
- Utilizar a lib [nodemailer](https://nodemailer.com/about/)) para envio de emails.

 ```
yarn add nodemailer
```

Serviços de envio de email:
	- SendGrid
	- Mailgun
	- Amazon SES (*Rocketseat Choice*)
	- Sparkpost
	- Mandril (Mailchimp)
	- Mailtrap (__DEV) // só para desenvolvimento

Vamos utilizar o Mailtrap para poder enviar o email para uma caixa de entrada fake, o email pode até ser verdadeiro, porém quando é enviado, ele não chega na caixa de entrada da pessoa, isso é ideal para termos em ambiente de desenvolvimento.

Primeiro passo é crair um conta no [https://mailtrap.io/](https://mailtrap.io/) e pegar as configurações.

Criar um arquivo de configuração de envio de emails:

`src/config/mail.js`:
```
export default {
  host: 'smtp.mailtrap.io',
  post: 2525,
  secure: false,
  auth: {
    user: '109b42360028f1',
    pass: '907f3523d2a604',
  },
  default: {
    from: 'Equipe GoBarber <noreply@gobarber.com>',
  },
};

```

Depois configurar o envio de email usando os dados do mail.js para se conectar ao provedor:

```
import nodemailer from 'nodemailer';
import mailConfig from '../config/mail';

class Mail {
  constructor() {
    const { host, port, secure, auth } = mailConfig;

    this.transporter = nodemailer.createTransport({
      host,
      port,
      secure,
      auth: auth.user ? auth : null,
    });
  }

  sendMail(message) {
    return this.transporter.sendMail({
      ...mailConfig.default,
      ...message,
    });
  }
}

export default new Mail();
```

Por fim utilizar no controller:

```
...
import Mail from  '../../lib/Mail';
...

// Alterando a busca de appointment para trazer o usuário:
const appointment = await Appointment.findByPk(req.params.id, {
      include: [
        {
          model: User,
          as: 'provider',
          attributes: ['name', 'email'],
        },
      ],
    });
...
// Enviando email
 await Mail.sendMail({
      to: `${appointment.provider.name} <${appointment.provider.email}>`,
      subject: 'Agendamento cancelado',
      text: 'Você tem um novo cancelamento',
   });
```

## Aula 33 - Configurando templates de email

- Criar templates de email, com html, css,
- Utilizar template engine: [https://handlebarsjs.com/](https://handlebarsjs.com/)
- Instalar libs para utilizar o handlebars:
	- `yarn add express-handlebars nodemailer-express-handlebars`

Basicamenteo foi feito a integração do handlebars no Mail.js para declarar onde estão as pasta de templates e a extensão de arquivo. E no `AppointmentController.js` foi utilizado o template `cancellation.hbs` no envio de email.

## Aula 34 - Configurando fila com Redis

Quando utilizamos rotas que enviam email para o usuário, elas estão demorando um pouco mais para concluir a requisição, pois elas esperam a requisição concluir para depois devolver a resposta para o usuário, e isso demora porque depende de um serviço externo de envio de email, que depende da internet, etc, fazendo com que demore para concluir a requisição.

Eu poderia remover o `async`e deixaria mais rápido a execução da requisição, pois o email seria enviado de forma assíncrona também, porém se desse algum problema no envio de email eu não poderia informar isso ao usuário.

```
   Mail.sendMail({
    ...
    });
```

A melhor forma de controlar isso é com FILAS, com background jobs, controlar que serviços rodem em segundo plano, e de forma que podemos enviar mensagem para o usuário.

Precismos de um banco não relacional que armazena chave e valor apenas, não tem schemas e nem models. Ele é muito mais perfomático. E iremos utilizar o Redis no Docker.

Para configurar o [https://redis.io/](https://redis.io/) no docker:

```
docker  run  --name redisbarber -p 6379:6379 -d  -t redis:alpine
```

A versão com alpine vem bem leve, vem com as features mais essencias do linux.

Agora vamos instalar o [bee-queue](https://github.com/bee-queue/bee-queue), que é uma ferramenta de background jobs no node, ele é mais simples e não tem todos os recursos que outros tem, por exemplo o [kue](https://github.com/Automattic/kue). Mas para essa aplicação já serve. Kue é menos performatico mas tem mais rebustez. Com Bee Queue ele agenda os jobs e faz retentativas de reenvio de email, que é o necessário e suficente para aplicação, por isso escolhemos essa lib.


Para instalar o bee-queue:

```
yarn add bee-queue
```

Passo a passo da codificação da configuração de envio de email com agendamento de jobs em filas:

- Primeiro configurar o Redis em execução da máquina com a aplicação:

`src/config/redis.js`:

```
export default {
  host: '127.0.0.1',
  port: 6379,
};
```
Coloco o endereço do servidor onde está executando o Redis e a porta de entrada de requisição.

- Depois crio o JOB: `CancellationMail.js` que é responsável pelo envio de email quando o usuário cancela um appointment, e removo o envio de email lá do `AppointmentController.js`:

```
import { format, parseISO } from 'date-fns';
import pt from 'date-fns/locale/pt';
import Mail from '../../lib/Mail';

class CancellationMail {
  get key() {
    return 'CancellationMail';
  }

  async handle({ data }) {
    const { appointment } = data;

    await Mail.sendMail({
      to: `${appointment.provider.name} <${appointment.provider.email}>`,
      subject: 'Agendamento cancelado',
      template: 'cancellation',
      context: {
        provider: appointment.provider.name,
        user: appointment.user.name,
        date: format(
          parseISO(appointment.date),
          "'dia' dd 'de' MMMM', às' H:mm'h'",
          {
            locale: pt,
          }
        ),
      },
    });
  }
}

export default new CancellationMail();
```

O Job precisa de uma chave (key) que é o nome do JOB, e o JOB em si, que é uma função que chamos de handle, que recebe vários parâmetros, e nesse caso desestruturamos o parâmetros para pegar apenas os dados (data) do appointment que foi enviado na criação do JOB que fica dentro da Queue que iremos ver no próximo código abaixo.
Depois o restante do código é o que estava no `AppointmentController.js` no método `delete`.


Logo, criamos a classe` Queue.js` que se refere a Fila, é a classe que faz o gerencimento da Fila, de inicializar, adicionar e processa fila.

```
import Bee from 'bee-queue';
import CancellationMail from '../app/jobs/CancellationMail';
import redisConfig from '../config/redis';

const jobs = [CancellationMail];

class Queue {
  constructor() {
    this.queues = {};
    this.init();
  }

  init() {
    jobs.forEach(({ key, handle }) => {
      this.queues[key] = {
        bee: new Bee(key, {
          redis: redisConfig,
        }),
        handle,
      };
    });
  }

  add(queue, job) {
    return this.queues[queue].bee.createJob(job).save();
  }

  processQueue() {
    jobs.forEach(job => {
      const { bee, handle } = this.queues[job.key];
      bee.process(handle);
    });
  }
}

export default new Queue();
```

Definimos uma const jobs, que recebe um array de jobs, no nosso caso só tem um Job, mas poderia ter mais.

No construtuor criamos uma váriavel `queues` que é um objeto que armazena os jobs na fila, e chamamos o `init()` para percorrer o array de jobs e adicionar na fila, a chave: 'CancellationMail' com o valor: que é um Bee que é uma instância do JOB que também recebe a mesma chave como valor e configuração de conexão com Redis, e por fim recebe o handle que é o método de dentro do job que nesse caso envia o email.

O método add, recebe a queue e o job. Queue é o nome do processo na fila e o job é o objeto que guarda os dados que serão executados no job. O método cria dentro da fila que foi inicializada o job em si, passando os valores que serão utilizados na execução do job, contém os dados do appointment no no caso.

O método processQueue percore os jobs, e chama a fila para ser processada passando o método handle que deve ser executado de dentro do Job.

Agora utilizamos o Job e fila Queue no AppointmentController.js:

Importando os códigos e substituindo o envio de email por Utililizar a instância de Queue para adicionar na fila o nome do Job e os dados que o Job deve processar:
```
...
import CancellationMail from  '../jobs/CancellationMail';
import Queue from  '../../lib/Queue';
...
await Queue.add(CancellationMail.key, { appointment });
...
```

Podemos deixar o job executando em uma outra instãncia do Node no servidor:
Criando um arquivo `src/queue.js`:
```
import Queue from  './lib/Queue';
Queue.processQueue();
```

Criar um script no package.json para rodar com sucrase uma vez que estamos utilizando o import/export:

```
"queue":  "nodemon src/queue.js"
```

E para executar só chamar: `yarn queue`

Para testar só tentar cancelar um appointment novamente.

## Aula 35 - Monitorando falhas na fila

Para poder ouvir os erros do processamento de jobs e obter um log, só alterar o código:

```
processQueue() {
    jobs.forEach(job => {
      const { bee, handle } = this.queues[job.key];
      bee.on('failed', this.handleFailure).process(handle);
    });
}

handleFailure(job, err) {
 console.log(`Queue ${job.queue.name}: FAILED`, err);
}
```

Com isso a cada erro que acontecer podemos ver no log do servidor, posteriomente iremos mostrar de forma mais amigável.

Na documentação do bee-queue tem outros listeners também, de sucesso, etc.


## Aula 36 - Listando horários disponíveis

Criar um controller para mostrar os horários disponíveis do prestador de serviço de uma dia
Quero saber todos os horários disponíveis do prestador para um determinado dia.

- Criar uma rota :
```
routes.get('/providers/:providerId/available', AvailableController.index);
```
Criar um controller `AvailableController.js`;

```
import {
  startOfDay,
  endOfDay,
  setHours,
  setMinutes,
  setSeconds,
  format,
  isAfter,
} from 'date-fns';
import { Op } from 'sequelize';
import Appointment from '../models/Appointment';

class AvailableController {
  async index(req, res) {
    const { date } = req.query;

    if (!date) {
      return res.status(400).json({ error: 'Invalid date' });
    }

    const searchDate = Number(date);

    // 2019-09-18 10:49:44

    const appointments = await Appointment.findAll({
      where: {
        provider_id: req.params.providerId,
        canceled_at: null,
        date: {
          [Op.between]: [startOfDay(searchDate), endOfDay(searchDate)],
        },
      },
    });

    const schedule = [
      '08:00', // 2019-09-18 08:00:00
      '09:00', // 2019-09-18 09:00:00
      '10:00', // 2019-09-18 10:00:00
      '11:00', // ...
      '12:00',
      '13:00',
      '14:00',
      '15:00',
      '16:00',
      '17:00',
      '18:00',
      '19:00',
    ];

    const available = schedule.map(time => {
      const [hour, minute] = time.split(':');
      const value = setSeconds(
        setMinutes(setHours(searchDate, hour), minute),
        0
      );

      return {
        time,
        // format to: 2019-09-18T15:40:44-04:00
        value: format(value, "yyyy-MM-dd'T'HH:mm:ssxxx"),
        available:
          isAfter(value, new Date()) &&
          !appointments.find(a => format(a.date, 'HH:mm') === time),
      };
    });

    return res.json(available);
  }
}

export default new AvailableController();
```

Só vai retornar os horários que não tem appointment marcado e que o valor desejado será posterior a data atual (isto é não se pode marcar um agendamento para um horário que já passou).

Recebo da requisição o ID do prestador e o dia que o usuário quer ver os horários disponíveis.

Essa data vem como timestamp e formato de string do componente datepicker do frontend. Então é preciso transformar um Number.

Depois busco todos agendamentos do provider informado pelo parâmetro da requisição, que não estejam cancelados, e que a data seja entre a primeira e última hora do dia informado.

Crio uma tabela estática de horários fixos.

E faço o restante da lógica e retorno para o usuário os horários em um objeto que retorna:

```
 {
    "time": "15:00",
    "value": "2019-09-18T15:00:00-04:00",
    "available": false
  },
  {
    "time": "16:00",
    "value": "2019-09-18T16:00:00-04:00",
    "available": true
  },
  {
    "time": "17:00",
    "value": "2019-09-18T17:00:00-04:00",
    "available": true
  },
```

## Aula 37 - Campos virtuais no agendamento

Listar campos a mais, para que o frontend possa mostrar um agendamento que já aconteceu, para isso vamos criar um variável past que será virtual, uma tabela que não existe na tabela, apenas no Model.

No `Appointment.js`:

```
import Sequelize, { Model } from 'sequelize';
import { isBefore, subHours } from 'date-fns';

class Appointment extends Model {
  static init(sequelize) {
    super.init(
      {
        date: Sequelize.DATE,
        canceled_at: Sequelize.DATE,
        past: {
          type: Sequelize.VIRTUAL,
          get() {
            return isBefore(this.date, new Date());
          },
        },
        cancelable: {
          type: Sequelize.VIRTUAL,
          get() {
            return isBefore(new Date(), subHours(this.date, 2));
          },
        },
      },
      {
        sequelize,
      }
    );

    return this;
  }

  static associate(models) {
    this.belongsTo(models.User, { foreignKey: 'user_id', as: 'user' });
    this.belongsTo(models.User, { foreignKey: 'provider_id', as: 'provider' });
  }
}

export default Appointment;
```

E no método `index` do `AppointmentController.js` passar os atributos `past` e `cancelable`:

```
...
attributes: ['id', 'date', 'past', 'cancelable'],
...
```

Agora no Insomnia só fazer a buscar e verificar se os campos são listados com `true` ou `false`.
