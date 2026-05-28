# 🟢 Desafio Júnior: Gerenciador de Produtos

Bem-vindo! Este é um desafio prático para você treinar a criação de uma **API REST** do zero. Não se preocupe se parecer muito no começo — o objetivo é aprender fazendo. 🚀

---

## 📚 O que você vai construir

Uma API para gerenciar o catálogo de produtos de um e-commerce. Administradores poderão criar, listar, atualizar e remover produtos, organizados por categorias.

---

## 🗂️ Modelo de Dados

Antes de escrever qualquer código, pense na relação entre as entidades:

- Uma **Categoria** pode ter vários **Produtos**
- Um **Produto** pertence a apenas uma **Categoria**

### Campos esperados

**Produto**

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `nome` | string | ✅ | Deve ser único |
| `descricao` | string | ❌ | Descrição livre |
| `preco` | número | ✅ | Maior que zero |
| `estoque` | inteiro | ✅ | Não pode ser negativo |
| `categoria_id` | número | ✅ | Deve existir no banco |

**Categoria**

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `nome` | string | ✅ | Deve ser único |

---

## 🔌 Endpoints

### Produtos

| Método | Endpoint | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| `POST` | `/produtos` | Cria um produto | `201` | `400` dados inválidos |
| `GET` | `/produtos` | Lista todos | `200` | — |
| `GET` | `/produtos/:id` | Busca por ID | `200` | `404` não encontrado |
| `PUT` | `/produtos/:id` | Atualiza | `200` | `404` / `400` |
| `DELETE` | `/produtos/:id` | Remove | `204` | `404` não encontrado |


> Lista vazia não é erro — retorne `200` com `[]`.

### Categorias

| Método | Endpoint | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| `POST` | `/categorias` | Cria uma categoria | `201` | `409` nome duplicado |
| `GET` | `/categorias` | Lista todas | `200` | — |
| `DELETE` | `/categorias/:id` | Remove | `204` | `404` / `400` com produtos vinculados |

> Não é permitido deletar uma categoria que ainda tem produtos. Retorne `400` com uma mensagem explicativa.

---

## ✅ Exemplos de requisição

### Criar categoria

```bash
curl -X POST http://localhost:8080/categorias \
  -H "Content-Type: application/json" \
  -d '{"nome": "Eletrônicos"}'
```

Resposta `201`:
```json
{
  "id": 1,
  "nome": "Eletrônicos",
  "criadoEm": "2026-05-21T10:00:00"
}
```

---

### Criar produto

```bash
curl -X POST http://localhost:8080/produtos \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Camiseta Azul",
    "descricao": "100% algodão, tamanho M",
    "preco": 49.90,
    "estoque": 100,
    "categoria_id": 1
  }'
```

Resposta `201`:
```json
{
  "id": 1,
  "nome": "Camiseta Azul",
  "descricao": "100% algodão, tamanho M",
  "preco": 49.90,
  "estoque": 100,
  "categoria": {
    "id": 1,
    "nome": "Eletrônicos"
  },
  "criadoEm": "2026-05-21T10:30:00",
  "atualizadoEm": "2026-05-21T10:30:00"
}
```

---

### Atualizar produto

O `PUT` substitui o recurso por completo — todos os campos devem ser enviados, inclusive os que não mudaram.

```bash
curl -X PUT http://localhost:8080/produtos/1 \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Camiseta Azul",
    "descricao": "100% algodão, tamanho G",
    "preco": 54.90,
    "estoque": 80,
    "categoria_id": 1
  }'
```

Resposta `200`:
```json
{
  "id": 1,
  "nome": "Camiseta Azul",
  "descricao": "100% algodão, tamanho G",
  "preco": 54.90,
  "estoque": 80,
  "categoria": {
    "id": 1,
    "nome": "Eletrônicos"
  },
  "criadoEm": "2026-05-21T10:30:00",
  "atualizadoEm": "2026-05-21T11:00:00"
}
```

> As mesmas validações do `POST` se aplicam aqui: `preco` deve ser maior que zero, `estoque` não pode ser negativo, `categoria_id` deve existir.

---

## ❌ Exemplos de erro

### Produto não encontrado — `404`

```json
{
  "timestamp": "2026-05-27T15:13:13.922031",
  "status": 404,
  "mensagem": "produto com id 1 nao encontrado"
}
```

### Categoria não cadastrada — `404`

```json
{
  "timestamp": "2026-05-27T15:13:13.922031",
  "status": 404,
  "mensagem": "Categoria com id 1 nao cadastrada"
}
```

### Nome de produto duplicado — `409`

```json
{
  "timestamp": "2026-05-27T15:15:15.979381",
  "status": 409,
  "mensagem": "Já existe um produto com o nome Camiseta Azul"
}
```

### Nome de categoria duplicado — `409`

```json
{
  "timestamp": "2026-05-27T15:16:05.153376",
  "status": 409,
  "mensagem": "Já existe uma categoria com o nome Eletrônicos"
}
```

### Erro de validação em campos — `400`

Quando um ou mais campos violam as regras, a resposta traz um array `erros` detalhando cada problema:

```json
{
  "timestamp": "2026-05-27T15:14:38.917029",
  "status": 400,
  "mensagem": "Erro de validação em campos",
  "erros": [
    {
      "campo": "preco",
      "mensagem": "preco tem que ser maior que zero"
    },
    {
      "campo": "estoque",
      "mensagem": "estoque nao pode ser negativo"
    }
  ]
}
```

---

## 🏗️ Organização do código

Além de fazer funcionar, capriche na organização — isso é o que separa um código que resolve do código que alguém consegue manter:

```
controller  →  recebe a requisição e devolve a resposta
service     →  aplica as regras de negócio
repository  →  faz as consultas no banco
```

Regras de negócio ficam no **service**, nunca no controller.

---

## ⭐ Bônus

Terminou tudo? Tente implementar também:

- **Atualizar a categoria de um produto no PUT** — envie um `categoria_id` diferente e o produto deve passar a pertencer à nova categoria. Se a categoria não existir, retorne `404` com mensagem clara.
- **Filtros no `GET /produtos`** — implemente os query params `categoria_id`, `preco_min` e `preco_max` para filtrar a listagem. Os filtros devem ser opcionais e combináveis entre si. Lista vazia não é erro — retorne `200` com `[]`.

---

## 🚀 Como entregar

1. Faça um fork do repositório
2. Crie sua solução em `solucoes/gerenciador-produtos/<sua-linguagem>/`
3. Inclua um `README.md` com instruções para rodar e exemplos de uso
4. Abra um Pull Request com o título: `[Solução] Gerenciador de Produtos - <Linguagem>`

---

## 💡 Dicas

- **Comece pelo modelo de dados** — entenda a relação entre `Produto` e `Categoria` antes de escrever qualquer código.
- **Valide antes de salvar** — cheque os dados recebidos antes de tentar qualquer operação no banco.
- **As mesmas regras valem no PUT** — não esqueça de validar `preco`, `estoque` e `categoria_id` também na atualização, não só na criação.
- **Lista vazia não é erro** — se nenhum produto for encontrado com os filtros aplicados, retorne `200` com `[]`, nunca `404`.
- **Não delete categoria com produtos** — tente remover primeiro e confira o comportamento.
- **Teste os cenários de erro** — tente criar um produto com `preco` negativo, com `categoria_id` inexistente, ou com nome duplicado. A API deve rejeitar com mensagem clara.
- **Docker é um diferencial** — se conseguir subir a aplicação e o banco com `docker compose up`, vai se destacar.

Boa sorte! Você consegue. 💙
