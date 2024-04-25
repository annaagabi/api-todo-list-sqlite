![header](https://64.media.tumblr.com/9f1a14a795b693c7aaf1b67aabdd75ee/89fde1ab0ef3e47c-85/s2048x3072/b2517d911a726aedc2023236185a298e7a133622.pnj)

# Tarefandoo - TODO List
Descrição... API - Back-End

:globe_with_meridians: API: https://todo-xfvj.onrender.com/


## 1. Criando:
**No terminal:**
- [x] Inicializando o projeto (criar o package. json): `npm init -y`
- [x] Baixar framework: `npm i express​`
- [x] Mudar e reiniciar o projeto automaticamente: `npm i nodemon`
- [x] Baixar o banco de dados: `npm i --save react-native-sqlite-storage`
- [x] Corrigir erros de servidor: `npm i cors`

## 2. Adicionar ao Script no Package.JSON:
- [x] Adicione ao package.json, na parte de “script” o: "start": "nodemon ./index.js localhost 3000"
```js
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "start": "nodemon ./index.js localhost 3000"
},
```

## 3. Criar o Banco de Dados
- [x] Para criar o banco, crie um arquivo chamado “sequelize.js” e adicione o seguinte trecho de código
```js
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize({
  dialect: 'sqlite',
  storage: './database.sqlite',
  logging: false // Desativa os logs do Sequelize
});

module.exports = sequelize;
```

## 4. Definição dos Tipos de Dados (Modelo) e Conexão com o Banco
- [x] Após a criação do banco deverá ser criado um arquivo que conterá o modelo de dados, neste caso é “todo.js” e então importar o banco de dados para conectar no modelo de dados
```js
const { DataTypes } = require('sequelize');
const sequelize = require('../sequelize.js');
```

- [x] Em seguida, deverá ser criado os atributos que serão inseridos e sua tipagem de dados
```js
const Todo = sequelize.define('Todo', {
  tarefa: {
    type: DataTypes.STRING,
    allowNull: false
  },
  desc: {
    type: DataTypes.STRING
  },
  categoria: {
    type: DataTypes.STRING
  },
  situacao: {
    type: DataTypes.BOOLEAN,
    defaultValue: false
  }
});


module.exports = Todo;
```

## 5. Iniciar o Banco e Conectar com a Porta de Serviço WEB
- [x] Importe as dependências e as inicialize
```js
const express = require('express');
const app = express();

const sequelize = require('./sequelize');

const cors = require('cors');
app.use(cors());

app.use(express.json());
```

- [x] Crie a rota principal para verificar se a API está funcionando
```js
const todoRoutes = require('./Routes/todoRoutes');
app.use('/todo', todoRoutes);

app.get('/', (req, res) => {
  res.json({ message: 'Testando API' });
});
```

- [x] Agora deverá criar uma função assíncrona para verificar se o banco de dados está funcionando
```js
async function initializeDatabase() {
    try {
      console.log('Iniciando sincronização do banco de dados...');
      await sequelize.sync();
      console.log('Banco de dados sincronizado com sucesso.');
    } catch (error) {
      console.error('Erro ao sincronizar o banco de dados:', error);
    }
}
 
initializeDatabase();
```

- [x] Defina a porta em que a aplicação irá rodar
```js
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
```

## 6 - Definição das Rotas Bases e Conexão com o Modelo de Dados

### 6.1. Importando
- [x] Inicie importando as dependencias e os modelos de dados
```js
const express = require('express');
const router = express.Router();
const Todo = require('../models/Todo.js');
```

### 6.2. Rota POST
- [x] Definição da rota para criação de dados
```js
router.post('/', async (req, res) => {
  const { tarefa, desc, categoria, situacao } = req.body;
 
 //Restrição para que seja obrigatório colocar o nome da tarefa
  if (!tarefa) {
    res.status(422).json({ error: 'O titulo da tarefa é obrigatório!' });
    return;
  }

  try {

    // Criando dados
    const todo = await Todo.create({
      tarefa,
      desc,
      categoria,
      situacao: situacao || false
    });

    // Em caso de êxito o recurso será criado com sucesso
    res.status(201).json({ message: 'Tarefa inserida no sistema com sucesso', todo });
  } catch (error) {

    // Em caso de erro irá exibir uma mensagem
    res.status(500).json({ error: error.message });
  }
});
```

### 6.3. Rota GET
- [x] Definição da rota para exibição todos os dados
```js
router.get('/', async (req, res) => {
  try {
    const todo = await Todo.findAll();

    // A requisição foi realizada com sucesso
    res.status(200).json(todo);

  } catch (error) {
    // Em caso de erro irá exibir uma mensagem
    res.status(500).json({ error: error.message });
  }
});
```

- [x] Definição da rota para exibição de dados especificos (serão "resgatados" pelo id)
```js
router.get('/:id', async (req, res) => {

  // Extrai o dado da requisição pela url = req.params
  const id = req.params.id;
 
  try {
    const todo = await Todo.findByPk(id);

    //Envia uma mesagem caso a tarefa não exista no banco de dados
    if (!todo) {
      res.status(422).json({ message: 'A tarefa não foi encontrada' });
      return;
    }

    // A requisição foi realizada com sucesso
    res.status(200).json(todo);

  } catch (error) {
    // Em caso de erro irá exibir uma mensagem
    res.status(500).json({ error: error.message });
  }
});
```

### 6.4. Rota PATCH
- [x] O PATCH faz uma atualização parcial dos dados
- [x] Definição da rota para atualização de dados específicos
```js
router.patch('/:id', async (req, res) => {

  // A url vai vir com o id da tarefa
  const id = req.params.id;
  const { tarefa, desc, categoria, situacao } = req.body;
 
  // O corpo vai vir com os dados que precisam ser atualizados
  const todoData = {
    tarefa,
    desc,
    categoria,
    situacao: situacao || false
  };
 
  try {
    const [updated] = await Todo.update(todoData, { where: { id } });
 
    // Se não fez a atualização
    if (updated === 0) {

      // Validação de existencia de post
      res.status(422).json({ message: 'A tarefa não foi encontrada' });
      return;
    }

    // A atualização foi realizada com sucesso
    res.status(200).json(todoData);

  } catch (error) {
    // Em caso de erro irá exibir uma mensagem
    res.status(500).json({ error: error.message });
  }
});
```

### 6.5. Rota DELETE
- [x] Definição da excluir dados específicos
```js
router.delete('/:id', async (req, res) => {

  // A url vai vir com o id da tarefa
  const id = req.params.id;
 
  try {
    const todo = await Todo.findByPk(id);

    // Validação de existencia de post
    if (!todo) {
      res.status(404).json({ message: 'A tarefa não foi encontrada' });
      return;
    }

    await todo.destroy();
    // A exclusão foi realizada com sucesso
    res.status(200).json({ message: 'A tarefa removida com sucesso' });


  } catch (error) {
    // Em caso de erro irá exibir uma mensagem
    res.status(500).json({ error: error.message });
  }
});
```

### 6.6. Exportar
- [x] Exportar o arquivo das rotas
```js
module.exports = router;
```

## 7. Endpoints
### 7.1 Para Verificar se a API Está Funcionando
- [x] {{URL}}/
- [x] https://todo-xfvj.onrender.com/

### 7.2 Para Acessar Todas as Tarefas
- [x] {{URL}}/todo
- [x] https://todo-xfvj.onrender.com/todo

### 7.3 Para Acessar, Deletar ou Atualizar uma Tarefa Especifica:
- [x] {{URL}}/todo/id
- [x] https://todo-xfvj.onrender.com/todo/1
