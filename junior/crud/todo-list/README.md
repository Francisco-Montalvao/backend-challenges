# To-Do List API

**Nível:** Júnior | **Tema:** CRUD | **Tempo estimado:** 2 a 3 dias

Bem-vindo! Este é um ótimo ponto de partida para praticar backend. Você vai construir uma API REST completa para gerenciar tarefas — um CRUD clássico que consolida seus conhecimentos sobre HTTP, banco de dados e organização de código.

## O que você vai construir

Uma startup quer um sistema simples para seus funcionários gerenciarem tarefas do dia a dia. Você vai criar a API que alimenta essa aplicação.

## Modelo de Dados

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `titulo` | texto | sim | máx. 100 caracteres |
| `descricao` | texto | não | — |
| `concluida` | booleano | — | padrão `false` |
| `criadaEm` | data | gerada auto | não pode ser alterada |

## Endpoints

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| POST | `/tarefas` | Criar tarefa | 201 | 400 |
| GET | `/tarefas` | Listar todas | 200 | — |
| GET | `/tarefas/{id}` | Buscar por ID | 200 | 404 |
| PUT | `/tarefas/{id}` | Atualizar | 200 | 400, 404 |
| DELETE | `/tarefas/{id}` | Remover | 204 | 404 |
| PATCH | `/tarefas/{id}/concluir` | Marcar como concluída | 200 | 404 |

## Exemplos de requisição

**Criar tarefa**

```bash
curl -X POST http://localhost:8080/tarefas \
  -H "Content-Type: application/json" \
  -d '{"titulo": "Estudar API REST", "descricao": "Ler a documentação do HTTP"}'
```

```json
// 201 Created
{
  "id": 1,
  "titulo": "Estudar API REST",
  "descricao": "Ler a documentação do HTTP",
  "concluida": false,
  "criadaEm": "2026-05-25"
}
```

**Buscar ID inexistente**

```bash
curl http://localhost:8080/tarefas/99
```

```json
// 404 Not Found
{ "erro": "Tarefa com id 99 não encontrada." }
```

## Qualidade do código

Organize em camadas:

```
controller  →  recebe a requisição e responde HTTP
service     →  aplica as regras de negócio
repository  →  acessa o banco de dados
```

Começar com tudo em um arquivo único está ok — reorganize quando sentir que está ficando difícil de ler.

## Critérios de avaliação

| Critério | Peso |
|---|---|
| Todos os endpoints funcionando corretamente | 40% |
| Validação e erros com status HTTP corretos | 25% |
| Organização do código | 20% |
| README explicando como rodar o projeto | 15% |

## Como entregar

1. Suba sua solução em um repositório público
2. Inclua um README no repositório explicando como rodar localmente
3. Abra uma [Discussion](https://github.com/Francisco-Montalvao/backend-challenges/discussions) com o link da sua solução

> Você escolhe o banco de dados — SQLite, PostgreSQL, MySQL. Banco em memória também funciona para começar.

## Dicas finais

- Valide os campos de entrada antes de salvar no banco
- Quando um ID não existe, retorne `404` — nunca `500`
- Status `204` não tem body — não retorne JSON no DELETE
- Testes automatizados e Docker são diferenciais, não requisitos

Boa sorte — vai ficar ótimo!
