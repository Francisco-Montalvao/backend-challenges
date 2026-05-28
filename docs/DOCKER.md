# 🐳 Docker Setup — Backend Challenges

Cada desafio inclui um `docker-compose.yml` pronto para usar. Aqui está como.

---

## 🚀 Início Rápido

### 1. Instale Docker

Se ainda não tem:
- [Docker Desktop](https://www.docker.com/products/docker-desktop) (Mac, Windows)
- [Docker + Docker Compose](https://docs.docker.com/engine/install/) (Linux)

### 2. Suba os Serviços

```bash
cd junior/crud/todo-list
docker-compose up
```

Isso irá:
- ✅ Baixar imagens necessárias
- ✅ Criar volumes de dados
- ✅ Subir banco + aplicação
- ✅ Exibir logs em tempo real

### 3. Acesse a API

```bash
curl http://localhost:3000/tarefas
```

### 4. Parar

```bash
docker-compose down

# Parar e remover volumes (dados)
docker-compose down -v
```

---

## 📋 Desafios com Docker

### 🟢 Júnior

| Desafio | Porta | Banco | Services |
|---------|-------|-------|----------|
| [To-Do List](./junior/crud/todo-list/) | 3000 | PostgreSQL | app + db + adminer |
| [Login JWT](./junior/autenticacao/login-jwt/) | 3000 | PostgreSQL | app + db |

### 🟡 Pleno

| Desafio | Porta | Broker | Services |
|---------|-------|--------|----------|
| [Fila de Notificações](./pleno/mensageria/fila-notificacoes/) | 3000 | RabbitMQ | app + db + rabbitmq |

### 🔴 Sênior

| Desafio | Portas | Services |
|---------|--------|----------|
| [Sistema de Pedidos](./senior/microsservicos/sistema-pedidos/) | 8000-3004 | 5 serviços + 5 BDs + RabbitMQ + Jaeger |

---

## 🔍 Inspecionando Dados

### PostgreSQL — Adminer (Júnior)

O `docker-compose.yml` da To-Do List inclui **Adminer**, uma UI para o banco.

```
http://localhost:8080

User: todo
Password: todo123
Database: todo_db
Server: db
```

### RabbitMQ — Management Panel (Pleno)

```
http://localhost:15672

User: guest
Password: guest
```

Veja filas, mensagens, conectando consumers.

### Jaeger — Distributed Tracing (Sênior)

```
http://localhost:16686
```

Rastreie requisições entre todos os 5 microsserviços.

---

## 🛠️ Troubleshooting

### Porta já em uso

```bash
# Descubra qual processo está usando a porta
lsof -i :3000

# Ou mude a porta no docker-compose.yml
ports:
  - "3001:3000"  # local:container
```

### Permissão negada no `/app/node_modules`

```bash
# Executar como root
docker-compose exec -u root app chown -R node:node /app

# Ou remover volume
docker-compose down -v
```

### Banco não inicializa

```bash
# Verifique logs
docker-compose logs db

# Recrie
docker-compose down -v
docker-compose up
```

### Conectar ao container

```bash
# Bash shell
docker-compose exec app bash

# Ver logs em tempo real
docker-compose logs -f app

# Executar comando único
docker-compose exec app npm test
```

---

## 📝 Customizar docker-compose.yml

### Mudar variáveis de ambiente

```yaml
environment:
  - PORT=3001         # Mude a porta
  - NODE_ENV=production
  - DB_POOL=20        # Conectões simultâneas
```

### Adicionar novo serviço

```yaml
services:
  redis:             # Novo serviço
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - todo-network
```

### Usar imagem local

```yaml
app:
  build:
    context: .
    dockerfile: ./Dockerfile.custom
```

---

## 🔐 Segurança em Produção

⚠️ **NUNCA use o `docker-compose.yml` de desenvolvimento como está em produção!**

### Coisas a fazer:

```yaml
# ✅ BOM
app:
  environment:
    - DB_PASSWORD=${DB_PASSWORD}  # Da variável de ambiente
  restart: unless-stopped          # Auto-reinicia
  resources:
    limits:
      memory: 512M                 # Limite de memória

# ❌ RUIM
app:
  environment:
    - DB_PASSWORD=senha123         # Hardcoded no arquivo!
```

---

## 💡 Tips & Tricks

### Rebuild após mudança de código

```bash
docker-compose up --build
```

### Em modo detached (background)

```bash
docker-compose up -d

# Ver status
docker-compose ps

# Seguir logs
docker-compose logs -f
```

### Compartilhar entre máquinas

```bash
# Exporte o estado
docker-compose config > docker-compose.prod.yml

# Em outra máquina
docker-compose -f docker-compose.prod.yml up
```

### Limpar espaço

```bash
# Remove containers/networks não usados
docker system prune

# Remove tudo (incluindo volumes)
docker system prune -a --volumes
```

---

## 📚 Próxima Etapa

Pronto com Docker localmente? Próximos passos:

1. **Kubernetes** — Orquestre múltiplos containers
2. **Docker Swarm** — Clustering simples
3. **CI/CD** — GitHub Actions, GitLab CI para deploy automático
4. **Registry Privado** — DockerHub, AWS ECR, Google Artifact Registry

---

**Desenvolvido com 💙 para backend-challenges**

