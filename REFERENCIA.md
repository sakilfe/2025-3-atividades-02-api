# Guia de Referência Rápida

## 🚀 Comandos Essenciais

### Configuração Inicial
```bash
# Instalar NestJS CLI globalmente
npm install -g @nestjs/cli

# Criar novo projeto
nest new tasks-api

# Instalar dependências do projeto
npm install @nestjs/typeorm typeorm sqlite3 class-validator class-transformer

# Iniciar em modo desenvolvimento
npm run start:dev
```

### Comandos de Desenvolvimento
```bash
# Desenvolvimento com hot reload
npm run start:dev

# Build do projeto
npm run build

# Produção
npm run start:prod

# Testes
npm run test

# Testes em modo watch
npm run test:watch

# Lint (verificação de código)
npm run lint
```

## 🔧 Estrutura de Arquivos Esperada

```
tasks-api/
├── src/
│   ├── app.module.ts          # Módulo principal
│   ├── main.ts                # Arquivo de inicialização
│   └── tasks/
│       ├── dto/
│       │   ├── create-task.dto.ts    # DTO para criação
│       │   └── update-task.dto.ts    # DTO para atualização
│       ├── task.entity.ts            # Entity do TypeORM
│       ├── tasks.controller.ts       # Controller REST
│       ├── tasks.module.ts           # Módulo de tarefas
│       └── tasks.service.ts          # Service com lógica de negócio
├── tasks.db                   # Banco SQLite (criado automaticamente)
└── package.json
```

## 📍 Endpoints da API

| Método | URL | Descrição | Status de Retorno |
|--------|-----|-----------|-------------------|
| `GET` | `/tasks` | Listar todas as tarefas | 200 OK |
| `GET` | `/tasks/:id` | Buscar tarefa por ID | 200 OK / 404 Not Found |
| `POST` | `/tasks` | Criar nova tarefa | 201 Created |
| `PUT` | `/tasks/:id` | Atualizar tarefa | 200 OK / 404 Not Found |
| `DELETE` | `/tasks/:id` | Deletar tarefa | 204 No Content / 404 Not Found |

## 🧪 Exemplos de Teste Rápido

### Criar Tarefa
```bash
curl -X POST http://localhost:3000/tasks \
  -H "Content-Type: application/json" \
  -d '{"title": "Teste", "description": "Descrição teste", "status": "aberto"}'
```

### Listar Tarefas
```bash
curl http://localhost:3000/tasks
```

### Buscar por ID
```bash
curl http://localhost:3000/tasks/1
```

### Atualizar Tarefa
```bash
curl -X PUT http://localhost:3000/tasks/1 \
  -H "Content-Type: application/json" \
  -d '{"status": "fazendo"}'
```

### Deletar Tarefa
```bash
curl -X DELETE http://localhost:3000/tasks/1
```

## 🛠️ Troubleshooting Comum

### Problema: "Cannot find module"
**Causa**: Dependências não instaladas
**Solução**:
```bash
npm install
```

### Problema: "Port 3000 is already in use"
**Causa**: Porta ocupada
**Soluções**:
```bash
# Matar processo na porta 3000
npx kill-port 3000

# Ou usar porta diferente
npm run start:dev -- --port 3001
```

### Problema: "Entity not found" 
**Causa**: Configuração incorreta no app.module.ts
**Solução**: Verificar caminho das entities:
```typescript
entities: [__dirname + '/**/*.entity{.ts,.js}'],
```

### Problema: Validação não funciona
**Causa**: ValidationPipe não configurado
**Solução**: Verificar main.ts:
```typescript
app.useGlobalPipes(new ValidationPipe({
  transform: true,
  whitelist: true,
  forbidNonWhitelisted: true,
}));
```

### Problema: CORS Error
**Causa**: CORS não habilitado
**Solução**: Verificar main.ts:
```typescript
app.enableCors({
  origin: true,
  credentials: true,
});
```

### Problema: "Cannot read property of undefined"
**Causa**: Service não injetado corretamente
**Solução**: Verificar constructor do controller:
```typescript
constructor(private readonly tasksService: TasksService) {}
```

## 📋 Status Codes HTTP

| Code | Significado | Quando Usar |
|------|-------------|-------------|
| 200 | OK | GET e PUT com sucesso |
| 201 | Created | POST com sucesso |
| 204 | No Content | DELETE com sucesso |
| 400 | Bad Request | Dados inválidos |
| 404 | Not Found | Recurso não encontrado |
| 500 | Internal Server Error | Erro do servidor |

## 🔍 Debugs Úteis

### Verificar se aplicação está rodando
```bash
curl http://localhost:3000/tasks
```

### Verificar logs detalhados
No main.ts, adicione:
```typescript
console.log('API rodando em http://localhost:3000');
```

### Verificar dados no banco SQLite
```bash
# Instalar sqlite3 CLI (se necessário)
npm install -g sqlite3

# Abrir banco
sqlite3 tasks.db

# Comandos SQLite úteis:
.tables          # Listar tabelas
.schema task     # Ver estrutura da tabela
SELECT * FROM task;  # Ver todos os dados
.exit            # Sair
```

## 📝 Validações dos DTOs

### CreateTaskDto
```typescript
{
  title: string (required, não vazio),
  description: string (required, não vazio),
  status: 'aberto' | 'fazendo' | 'finalizado' (opcional)
}
```

### UpdateTaskDto  
```typescript
{
  title?: string (opcional),
  description?: string (opcional), 
  status?: 'aberto' | 'fazendo' | 'finalizado' (opcional)
}
```

## 🎯 Checklist Rápido de Entrega

- [ ] Aplicação inicia sem erros
- [ ] Todos os 5 endpoints funcionam
- [ ] Validações implementadas
- [ ] README.md completo
- [ ] Código no GitHub
- [ ] Banco de dados com dados de exemplo

## 📞 Contatos e Recursos

- **Documentação NestJS**: https://docs.nestjs.com/
- **TypeORM Docs**: https://typeorm.io/
- **Class Validator**: https://github.com/typestack/class-validator
- **HTTP Status Codes**: https://httpstatuses.com/

## 🆘 Comandos de Emergência

```bash
# Limpar cache do npm
npm cache clean --force

# Reinstalar dependências
rm -rf node_modules package-lock.json
npm install

# Reset do git (cuidado!)
git reset --hard HEAD

# Ver processos usando porta 3000
lsof -i :3000

# Matar processo específico
kill -9 <PID>
```

## 💡 Dicas de Produtividade

1. **Use auto-save** no VS Code
2. **Configure snippets** para código repetitivo
3. **Use Thunder Client** no VS Code para testar APIs
4. **Ative o Prettier** para formatação automática
5. **Use Git commits frequentes** com mensagens descritivas

---

**Lembre-se**: Se algo não funcionar, sempre verifique:
1. Dependências instaladas (`npm install`)
2. Aplicação rodando (`npm run start:dev`)
3. Porta correta (3000)
4. Sintaxe dos JSONs
5. Headers das requisições (`Content-Type: application/json`)