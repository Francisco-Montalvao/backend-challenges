# 🟢 Desafio Júnior: To-Do List API

**Nível:** Júnior | **Tema:** CRUD | **Tempo estimado:** 2 a 3 dias

Bem-vindo! Este é um ótimo ponto de partida para praticar backend. Você vai construir uma API REST completa para gerenciar tarefas — um CRUD clássico que consolida seus conhecimentos sobre HTTP, banco de dados e organização de código.

---

## 📚 O que você vai construir

Uma startup quer um sistema simples para seus funcionários gerenciarem tarefas do dia a dia. Você vai criar a API que alimenta essa aplicação.

---

## 🗂️ Modelo de Dados

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `titulo` | texto | ✅ | máx. 100 caracteres |
| `descricao` | texto | ❌ | — |
| `concluida` | booleano | — | padrão `false` |
| `criadaEm` | data | gerada auto | não pode ser alterada |

---

## 🗄️ Banco de Dados

Você escolhe o banco — o desafio não impõe nenhum. Algumas opções:

| Banco | Quando usar |
|---|---|
| **H2 / SQLite** | Quer começar rápido sem instalar nada |
| **PostgreSQL** | Quer simular um ambiente mais próximo do mercado |
| **MySQL** | Já tem familiaridade com ele |

> Banco em memória funciona para começar e facilita o setup. Quando se sentir confortável, vale migrar para um banco persistente.

---

## 🔌 Endpoints

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| `POST` | `/tarefas` | Criar tarefa | `201` | `400` |
| `GET` | `/tarefas` | Listar todas | `200` | — |
| `GET` | `/tarefas/{id}` | Buscar por ID | `200` | `404` |
| `DELETE` | `/tarefas/{id}` | Remover | `204` | `404` |
| `PATCH` | `/tarefas/{id}/concluir` | Marcar como concluída | `200` | `404` |

> Não há endpoint de atualização — a única transição de estado de uma tarefa é concluída ou não concluída, coberta pelo `PATCH`. Se precisar corrigir uma tarefa, delete e crie novamente.

---

## ✅ Exemplos de requisição

### Criar tarefa

```bash
curl -X POST http://localhost:8080/tarefas \
  -H "Content-Type: application/json" \
  -d '{"titulo": "Estudar API REST", "descricao": "Ler a documentação do HTTP"}'
```

Resposta `201`:
```json
{
  "id": 1,
  "titulo": "Estudar API REST",
  "descricao": "Ler a documentação do HTTP",
  "concluida": false,
  "criadaEm": "2026-05-25"
}
```

### Marcar como concluída

```bash
curl -X PATCH http://localhost:8080/tarefas/1/concluir
```

Resposta `200`:
```json
{
  "id": 1,
  "titulo": "Estudar API REST",
  "descricao": "Ler a documentação do HTTP",
  "concluida": true,
  "criadaEm": "2026-05-25"
}
```

---

## ❌ Exemplos de erro

### Tarefa não encontrada — `404`

```json
{
  "timestamp": "2026-05-27T15:13:13.922031",
  "status": 404,
  "mensagem": "Tarefa com id 99 não encontrada"
}
```

### Erro de validação em campos — `400`

```json
{
  "timestamp": "2026-05-27T15:14:38.917029",
  "status": 400,
  "mensagem": "Erro de validação em campos",
  "erros": [
    {
      "campo": "titulo",
      "mensagem": "Titulo não pode estar vazio"
    },
    {
      "campo": "titulo",
      "mensagem": "Titulo deve ter no máximo 100 caracteres"
    }
  ]
}
```

---

## 🏗️ Organização do código

Organize em camadas — a estrutura vai variar conforme a linguagem, mas uma referência possível é:

```
controller  →  recebe a requisição e responde HTTP
service     →  aplica as regras de negócio
repository  →  acessa o banco de dados
model       →  representa a entidade no banco
dto         →  objetos de entrada e saída da API
exception   →  exceções customizadas e handler global
```

Começar com tudo em um arquivo único está ok — reorganize quando sentir que está ficando difícil de ler.

---

## 🚀 Como entregar

Consulte o [CONTRIBUTING.md](../../CONTRIBUTING.md) para ver como compartilhar sua solução.

---

## 💡 Dicas

- **Valide os campos de entrada** antes de salvar no banco
- **Quando um ID não existe, retorne `404`** — nunca `500`
- **Status `204` não tem body** — não retorne JSON no DELETE
- **PATCH sem body** — o endpoint `/concluir` não recebe nada no body, só o `id` na URL
- **Testes automatizados e Docker são diferenciais**, não requisitos

Boa sorte — vai ficar ótimo! 💙
