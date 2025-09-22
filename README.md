# Atividade Prática: Desenvolvimento de API REST com NestJS

## Descrição da Atividade
Esta atividade prática tem como objetivo consolidar os conhecimentos adquiridos no [tutorial introdutório de API REST e NestJS](https://github.com/infoweb-pos/api-nest-notas-01-introducao) através da implementação de um CRUD completo para gerenciamento de tarefas.

## Objetivos de Aprendizado
Ao final desta atividade, o aluno será capaz de:
- Configurar um projeto NestJS do zero
- Implementar um CRUD completo com TypeORM
- Configurar banco de dados SQLite3
- Aplicar validações com Class Validator
- Testar endpoints de uma API REST
- Implementar boas práticas de desenvolvimento com NestJS

## Pré-requisitos
Antes de iniciar a atividade, certifique-se de ter:
- [ ] **Node.js** (versão 18.x ou superior) instalado
- [ ] **npm** ou **yarn** como gerenciador de pacotes
- [ ] **Git** instalado e configurado
- [ ] **Editor de código** (VS Code recomendado)
- [ ] **Cliente REST** para testes (Postman, Insomnia, ou Thunder Client)

### Verificação dos Pré-requisitos
Execute os comandos abaixo para verificar se os pré-requisitos estão instalados:

```bash
# Verificar versão do Node.js
node --version

# Verificar versão do npm
npm --version

# Verificar versão do Git
git --version
```

## Passos da Atividade

### Passo 1: Fork e Clone do Repositório Base
1. Acesse o repositório do tutorial: https://github.com/infoweb-pos/api-nest-notas-01-introducao
2. Clique no botão **Fork** no canto superior direito para criar uma cópia em sua conta
3. Clone seu fork para sua máquina local:
```bash
git clone https://github.com/SEU_USUARIO/api-nest-notas-01-introducao.git
cd api-nest-notas-01-introducao
```
4. Adicione o repositório original como upstream (opcional, mas recomendado):
```bash
git remote add upstream https://github.com/infoweb-pos/api-nest-notas-01-introducao.git
```

### Passo 2: Estudo do Material Teórico
1. Leia completamente o arquivo README.md do repositório clonado
2. Compreenda os conceitos de:
   - API REST e seus princípios
   - Métodos HTTP (GET, POST, PUT, DELETE)
   - Códigos de status HTTP
   - Estrutura de URLs REST
   - Conceitos do NestJS (Controllers, Services, Modules, DTOs, Entities)

### Passo 3: Configuração do Ambiente de Desenvolvimento

#### 3.1 Instalação Global do NestJS CLI
```bash
npm install -g @nestjs/cli
```

#### 3.2 Criação do Projeto
```bash
# Criar novo projeto NestJS
nest new tasks-api

# Navegar para o diretório do projeto
cd tasks-api

# Instalar dependências adicionais
npm install @nestjs/typeorm typeorm sqlite3 class-validator class-transformer
```

### Passo 4: Configuração do Banco de Dados SQLite

#### 4.1 Configurar TypeORM no app.module.ts
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { TasksModule } from './tasks/tasks.module';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'sqlite',
      database: 'tasks.db',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true, // Apenas para desenvolvimento
    }),
    TasksModule,
  ],
})
export class AppModule {}
```

### Passo 5: Implementação da Entity Task

#### 5.1 Criar a estrutura de diretórios
```bash
mkdir src/tasks
mkdir src/tasks/dto
```

#### 5.2 Implementar task.entity.ts
```typescript
import { Entity, Column, PrimaryGeneratedColumn, CreateDateColumn, UpdateDateColumn } from 'typeorm';

export enum TaskStatus {
  ABERTO = 'aberto',
  FAZENDO = 'fazendo',
  FINALIZADO = 'finalizado',
}

@Entity()
export class Task {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  title: string;

  @Column()
  description: string;

  @Column({
    type: 'text',
    enum: TaskStatus,
    default: TaskStatus.ABERTO,
  })
  status: TaskStatus;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

### Passo 6: Implementação dos DTOs

#### 6.1 Criar create-task.dto.ts
```typescript
import { IsString, IsEnum, IsNotEmpty, IsOptional } from 'class-validator';
import { TaskStatus } from '../task.entity';

export class CreateTaskDto {
  @IsString()
  @IsNotEmpty()
  title: string;

  @IsString()
  @IsNotEmpty()
  description: string;

  @IsEnum(TaskStatus)
  @IsOptional()
  status?: TaskStatus;
}
```

#### 6.2 Criar update-task.dto.ts
```typescript
import { IsString, IsEnum, IsOptional } from 'class-validator';
import { TaskStatus } from '../task.entity';

export class UpdateTaskDto {
  @IsString()
  @IsOptional()
  title?: string;

  @IsString()
  @IsOptional()
  description?: string;

  @IsEnum(TaskStatus)
  @IsOptional()
  status?: TaskStatus;
}
```

### Passo 7: Implementação do Service

#### 7.1 Criar tasks.service.ts
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Task } from './task.entity';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';

@Injectable()
export class TasksService {
  constructor(
    @InjectRepository(Task)
    private taskRepository: Repository<Task>,
  ) {}

  async findAll(): Promise<Task[]> {
    return this.taskRepository.find();
  }

  async findOne(id: number): Promise<Task> {
    const task = await this.taskRepository.findOne({ where: { id } });
    if (!task) {
      throw new NotFoundException(`Tarefa com ID ${id} não encontrada`);
    }
    return task;
  }

  async create(createTaskDto: CreateTaskDto): Promise<Task> {
    const task = this.taskRepository.create(createTaskDto);
    return this.taskRepository.save(task);
  }

  async update(id: number, updateTaskDto: UpdateTaskDto): Promise<Task> {
    const task = await this.findOne(id);
    Object.assign(task, updateTaskDto);
    return this.taskRepository.save(task);
  }

  async remove(id: number): Promise<void> {
    const task = await this.findOne(id);
    await this.taskRepository.remove(task);
  }
}
```

### Passo 8: Implementação do Controller

#### 8.1 Criar tasks.controller.ts
```typescript
import {
  Controller,
  Get,
  Post,
  Body,
  Put,
  Param,
  Delete,
  ParseIntPipe,
  HttpCode,
  HttpStatus,
} from '@nestjs/common';
import { TasksService } from './tasks.service';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';
import { Task } from './task.entity';

@Controller('tasks')
export class TasksController {
  constructor(private readonly tasksService: TasksService) {}

  @Get()
  findAll(): Promise<Task[]> {
    return this.tasksService.findAll();
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number): Promise<Task> {
    return this.tasksService.findOne(id);
  }

  @Post()
  @HttpCode(HttpStatus.CREATED)
  create(@Body() createTaskDto: CreateTaskDto): Promise<Task> {
    return this.tasksService.create(createTaskDto);
  }

  @Put(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() updateTaskDto: UpdateTaskDto,
  ): Promise<Task> {
    return this.tasksService.update(id, updateTaskDto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  async remove(@Param('id', ParseIntPipe) id: number): Promise<void> {
    return this.tasksService.remove(id);
  }
}
```

### Passo 9: Configuração do Module

#### 9.1 Criar tasks.module.ts
```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { TasksController } from './tasks.controller';
import { TasksService } from './tasks.service';
import { Task } from './task.entity';

@Module({
  imports: [TypeOrmModule.forFeature([Task])],
  controllers: [TasksController],
  providers: [TasksService],
})
export class TasksModule {}
```

### Passo 10: Configuração Final

#### 10.1 Atualizar main.ts
```typescript
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  
  // Habilitar CORS
  app.enableCors({
    origin: true,
    credentials: true,
  });

  // Habilitar validação global
  app.useGlobalPipes(new ValidationPipe({
    transform: true,
    whitelist: true,
    forbidNonWhitelisted: true,
  }));

  await app.listen(3000);
  console.log('API rodando em http://localhost:3000');
}
bootstrap();
```

### Passo 11: Execução e Testes

#### 11.1 Iniciar a aplicação
```bash
npm run start:dev
```

#### 11.2 Testar os endpoints

**1. Criar tarefa (POST /tasks)**
```bash
curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Estudar NestJS",
    "description": "Aprender conceitos básicos do framework",
    "status": "aberto"
  }'
```

**2. Listar tarefas (GET /tasks)**
```bash
curl http://localhost:3000/tasks
```

**3. Buscar tarefa específica (GET /tasks/:id)**
```bash
curl http://localhost:3000/tasks/1
```

**4. Atualizar tarefa (PUT /tasks/:id)**
```bash
curl -X PUT http://localhost:3000/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{
    "status": "fazendo"
  }'
```

**5. Deletar tarefa (DELETE /tasks/:id)**
```bash
curl -X DELETE http://localhost:3000/tasks/1
```

### Passo 12: Validação dos Resultados

#### 12.1 Verificar estrutura final de arquivos
```
src/
├── app.module.ts
├── main.ts
└── tasks/
    ├── dto/
    │   ├── create-task.dto.ts
    │   └── update-task.dto.ts
    ├── task.entity.ts
    ├── tasks.controller.ts
    ├── tasks.module.ts
    └── tasks.service.ts
```

#### 12.2 Testar todos os cenários
- [ ] Criar tarefas com status diferentes
- [ ] Validar que campos obrigatórios são respeitados
- [ ] Testar busca por ID inexistente (deve retornar 404)
- [ ] Testar atualização parcial de tarefas
- [ ] Verificar se timestamps são criados automaticamente

## Entregáveis

### Entrega Obrigatória
1. **Repositório Git** com o código fonte completo
2. **README.md** documentando:
   - Como executar o projeto
   - Endpoints disponíveis
   - Exemplos de uso de cada endpoint
3. **Arquivo de banco SQLite** (tasks.db) com dados de exemplo
4. **Screenshots ou vídeo** demonstrando os endpoints funcionando

### Entrega Opcional (Para Nota Extra)
1. **Testes unitários** para o service
2. **Documentação Swagger** configurada
3. **Docker** configurado para execução em container
4. **Middleware personalizado** para logging de requisições

## Critérios de Avaliação

### Funcionalidades Básicas (70 pontos)
- [ ] **Configuração correta do projeto NestJS** (10 pts)
- [ ] **Entity Task implementada corretamente** (10 pts)
- [ ] **DTOs com validação adequada** (10 pts)
- [ ] **Service com todos os métodos CRUD** (15 pts)
- [ ] **Controller com todos os endpoints** (15 pts)
- [ ] **Aplicação executa sem erros** (10 pts)

### Boas Práticas (20 pontos)
- [ ] **Estrutura de pastas organizada** (5 pts)
- [ ] **Código limpo e bem comentado** (5 pts)
- [ ] **Tratamento de erros adequado** (5 pts)
- [ ] **Uso correto dos decorators NestJS** (5 pts)

### Documentação (10 pontos)
- [ ] **README.md completo** (5 pts)
- [ ] **Exemplos de uso dos endpoints** (5 pts)

## Troubleshooting

### Problemas Comuns

#### Erro: "Cannot find module"
**Solução:** Verifique se todas as dependências foram instaladas:
```bash
npm install
```

#### Erro: "Entity not found"
**Solução:** Certifique-se de que o caminho das entities está correto no app.module.ts:
```typescript
entities: [__dirname + '/**/*.entity{.ts,.js}'],
```

#### Erro de validação não funciona
**Solução:** Verifique se o ValidationPipe está configurado no main.ts:
```typescript
app.useGlobalPipes(new ValidationPipe({
  transform: true,
  whitelist: true,
  forbidNonWhitelisted: true,
}));
```

#### Banco de dados não é criado
**Solução:** Certifique-se de que `synchronize: true` está configurado no TypeORM (apenas para desenvolvimento).

### Dicas Importantes
1. **Sempre execute `npm run start:dev`** para desenvolvimento com hot reload
2. **Use ferramentas de cliente REST** como Postman ou Insomnia para testes mais fáceis
3. **Verifique logs no console** para identificar problemas
4. **Faça commits frequentes** durante o desenvolvimento
5. **Teste cada endpoint** após implementar

## Recursos Adicionais
- [Documentação Oficial NestJS](https://docs.nestjs.com/)
- [TypeORM Documentation](https://typeorm.io/)
- [Class Validator Documentation](https://github.com/typestack/class-validator)
- [REST API Best Practices](https://restfulapi.net/)

## Prazo de Entrega
**Data limite:** [A ser definido pelo professor]

**Forma de entrega:** Link do repositório GitHub com o código fonte completo.

---

*Esta atividade é baseada no tutorial disponível em: https://github.com/infoweb-pos/api-nest-notas-01-introducao*
