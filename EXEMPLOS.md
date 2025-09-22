# Exemplos Práticos e Recursos Adicionais

## Exemplos de Requisições HTTP

### 1. Testando com cURL

#### Criar uma nova tarefa
```bash
curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Implementar autenticação",
    "description": "Adicionar JWT ao sistema de login",
    "status": "aberto"
  }'
```

**Resposta esperada (201 Created):**
```json
{
  "id": 1,
  "title": "Implementar autenticação",
  "description": "Adicionar JWT ao sistema de login",
  "status": "aberto",
  "createdAt": "2024-01-15T10:30:00.000Z",
  "updatedAt": "2024-01-15T10:30:00.000Z"
}
```

#### Listar todas as tarefas
```bash
curl http://localhost:3000/tasks
```

**Resposta esperada (200 OK):**
```json
[
  {
    "id": 1,
    "title": "Implementar autenticação",
    "description": "Adicionar JWT ao sistema de login",
    "status": "aberto",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  },
  {
    "id": 2,
    "title": "Criar testes unitários",
    "description": "Adicionar cobertura de testes",
    "status": "fazendo",
    "createdAt": "2024-01-15T11:00:00.000Z",
    "updatedAt": "2024-01-15T11:00:00.000Z"
  }
]
```

#### Buscar tarefa por ID
```bash
curl http://localhost:3000/tasks/1
```

#### Atualizar tarefa
```bash
curl -X PUT http://localhost:3000/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "status": "fazendo",
    "description": "Adicionar JWT ao sistema de login - Em andamento"
  }'
```

#### Deletar tarefa
```bash
curl -X DELETE http://localhost:3000/tasks/1
```

**Resposta esperada:** Status 204 No Content (sem corpo de resposta)

### 2. Testando com JavaScript (Fetch API)

```javascript
// Função para criar uma tarefa
async function criarTarefa() {
  const response = await fetch('http://localhost:3000/tasks', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      title: 'Nova tarefa via JS',
      description: 'Testando com JavaScript',
      status: 'aberto'
    })
  });
  
  const data = await response.json();
  console.log('Tarefa criada:', data);
}

// Função para listar tarefas
async function listarTarefas() {
  const response = await fetch('http://localhost:3000/tasks');
  const data = await response.json();
  console.log('Lista de tarefas:', data);
}

// Função para atualizar tarefa
async function atualizarTarefa(id, dadosAtualizacao) {
  const response = await fetch(`http://localhost:3000/tasks/${id}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(dadosAtualizacao)
  });
  
  const data = await response.json();
  console.log('Tarefa atualizada:', data);
}
```

## Exemplos de Validação

### Casos de Sucesso

**Entrada válida:**
```json
{
  "title": "Tarefa válida",
  "description": "Descrição da tarefa",
  "status": "aberto"
}
```

### Casos de Erro

**Título vazio (400 Bad Request):**
```json
{
  "title": "",
  "description": "Descrição válida"
}
```

**Resposta de erro:**
```json
{
  "statusCode": 400,
  "message": [
    "title should not be empty"
  ],
  "error": "Bad Request"
}
```

**Status inválido (400 Bad Request):**
```json
{
  "title": "Título válido",
  "description": "Descrição válida",
  "status": "status_invalido"
}
```

**Resposta de erro:**
```json
{
  "statusCode": 400,
  "message": [
    "status must be one of the following values: aberto, fazendo, finalizado"
  ],
  "error": "Bad Request"
}
```

## Script de População do Banco

Crie um arquivo `populate-db.js` na raiz do projeto para popular o banco com dados de exemplo:

```javascript
const axios = require('axios');

const tarefasExemplo = [
  {
    title: 'Estudar NestJS',
    description: 'Completar tutorial básico do NestJS',
    status: 'fazendo'
  },
  {
    title: 'Implementar CRUD',
    description: 'Criar operações básicas de Create, Read, Update, Delete',
    status: 'finalizado'
  },
  {
    title: 'Configurar banco de dados',
    description: 'Configurar TypeORM com SQLite',
    status: 'finalizado'
  },
  {
    title: 'Adicionar validações',
    description: 'Implementar validações com class-validator',
    status: 'aberto'
  },
  {
    title: 'Criar testes',
    description: 'Escrever testes unitários para o service',
    status: 'aberto'
  }
];

async function popularBanco() {
  console.log('Iniciando população do banco de dados...');
  
  for (let i = 0; i < tarefasExemplo.length; i++) {
    try {
      const response = await axios.post('http://localhost:3000/tasks', tarefasExemplo[i]);
      console.log(`Tarefa ${i + 1} criada:`, response.data.title);
    } catch (error) {
      console.error(`Erro ao criar tarefa ${i + 1}:`, error.message);
    }
  }
  
  console.log('População do banco concluída!');
}

popularBanco();
```

Para usar o script:
```bash
npm install axios
node populate-db.js
```

## Estrutura Completa de Arquivos

```
tasks-api/
├── src/
│   ├── app.module.ts
│   ├── main.ts
│   └── tasks/
│       ├── dto/
│       │   ├── create-task.dto.ts
│       │   └── update-task.dto.ts
│       ├── task.entity.ts
│       ├── tasks.controller.ts
│       ├── tasks.module.ts
│       └── tasks.service.ts
├── test/
├── node_modules/
├── tasks.db (criado automaticamente)
├── populate-db.js (opcional)
├── package.json
├── package-lock.json
├── tsconfig.json
├── nest-cli.json
└── README.md
```

## Comandos Úteis para Desenvolvimento

```bash
# Iniciar em modo de desenvolvimento (com hot reload)
npm run start:dev

# Construir o projeto
npm run build

# Executar em modo de produção
npm run start:prod

# Executar testes
npm run test

# Executar testes em modo watch
npm run test:watch

# Gerar cobertura de testes
npm run test:cov

# Verificar sintaxe (lint)
npm run lint

# Corrigir problemas de sintaxe automaticamente
npm run lint:fix
```

## Extensões Recomendadas para VS Code

```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "ms-vscode.vscode-json",
    "rangav.vscode-thunder-client",
    "formulahendry.auto-rename-tag",
    "christian-kohler.path-intellisense"
  ]
}
```

## Configuração do Prettier (.prettierrc)

```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 80,
  "tabWidth": 2,
  "useTabs": false
}
```

## Próximos Passos Avançados

### 1. Implementar Paginação
```typescript
// No controller
@Get()
async findAll(
  @Query('page') page: number = 1,
  @Query('limit') limit: number = 10,
) {
  return this.tasksService.findAll(page, limit);
}

// No service
async findAll(page: number = 1, limit: number = 10) {
  const skip = (page - 1) * limit;
  return this.taskRepository.find({
    skip,
    take: limit,
  });
}
```

### 2. Adicionar Filtros
```typescript
// DTO para filtros
export class FilterTaskDto {
  @IsOptional()
  @IsEnum(TaskStatus)
  status?: TaskStatus;

  @IsOptional()
  @IsString()
  search?: string;
}

// No service
async findWithFilters(filters: FilterTaskDto) {
  const query = this.taskRepository.createQueryBuilder('task');
  
  if (filters.status) {
    query.andWhere('task.status = :status', { status: filters.status });
  }
  
  if (filters.search) {
    query.andWhere('task.title LIKE :search OR task.description LIKE :search', 
      { search: `%${filters.search}%` });
  }
  
  return query.getMany();
}
```

### 3. Implementar Swagger
```bash
npm install @nestjs/swagger swagger-ui-express
```

```typescript
// No main.ts
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

const config = new DocumentBuilder()
  .setTitle('Tasks API')
  .setDescription('API para gerenciamento de tarefas')
  .setVersion('1.0')
  .build();
  
const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api', app, document);
```

## Glossário de Termos

- **API**: Interface de Programação de Aplicações
- **REST**: Representational State Transfer
- **CRUD**: Create, Read, Update, Delete
- **DTO**: Data Transfer Object
- **Entity**: Representação de uma tabela do banco de dados
- **Repository**: Padrão para acesso a dados
- **Dependency Injection**: Injeção de Dependência
- **Decorator**: Anotação que adiciona metadados a classes/métodos
- **ORM**: Object-Relational Mapping
- **Hot Reload**: Recarga automática durante desenvolvimento