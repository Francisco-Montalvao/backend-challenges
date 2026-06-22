# 🟡 Desafio Junior: Gestão de Pedidos

**Nível:** Junior++ | **Tema:** CRUD + Relacionamentos + Relatórios | **Tempo estimado:** 5 a 7 dias

Você já sabe fazer CRUD. Agora o jogo muda. Este desafio testa sua capacidade de modelar um sistema com múltiplas entidades relacionadas, aplicar regras de negócio em transições de estado e extrair informações consolidadas do banco usando agregações SQL.

---

## 📚 O que você vai construir

Uma loja online precisa de uma API para gerenciar seus pedidos. O sistema deve permitir o cadastro de clientes, produtos organizados por categorias, e o registro de pedidos com múltiplos itens. Além disso, o time de negócio precisa de relatórios para acompanhar a performance das vendas.

---

## 🧭 Convenções

Antes de mergulhar nos endpoints, alinhe estes pontos — eles valem para **toda** a API.

### Status HTTP

| Código | Significado | Quando usar |
|---|---|---|
| `200 OK` | Sucesso com corpo | GET, PUT e PATCH bem-sucedidos |
| `201 Created` | Recurso criado | POST que cria um recurso |
| `204 No Content` | Sucesso sem corpo | DELETE, inativação e cancelamento |
| `400 Bad Request` | Validação ou regra de negócio violada | Campos inválidos, transição inválida, estoque, etc. |
| `404 Not Found` | Recurso não encontrado | ID inexistente |
| `409 Conflict` | Conflito de unicidade | E-mail ou nome já cadastrado |

### Formato de erro padrão

Erros de regra de negócio ou recurso não encontrado seguem este formato:

```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "Descrição clara e legível do erro."
}
```

### Formato de erro de validação de campos

Quando um ou mais campos do corpo são inválidos, retorne `400` com a lista de erros:

```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "Erro de validação em campos",
  "erros": [
    { "campo": "nome", "mensagem": "O nome deve ter no mínimo 2 caracteres." },
    { "campo": "email", "mensagem": "O e-mail informado é inválido." }
  ]
}
```

### Outros padrões

- **Nomenclatura:** todos os campos JSON usam `snake_case` (ex.: `valor_total`, `criado_em`, `preco_unitario`).
- **Datas e horários:** formato ISO-8601 (`YYYY-MM-DDTHH:mm:ss`); datas puras em `YYYY-MM-DD`.
- **Valores monetários:** decimal com 2 casas (ex.: `89.90`).

---

## 🗂️ Modelo de Dados

Antes de escrever qualquer código, modele as relações entre as entidades. Entender os relacionamentos é fundamental para construir as queries de relatório corretamente.

**clientes**

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `nome` | texto | ✅ | Mínimo 2 caracteres |
| `email` | texto | ✅ | Formato válido, único no banco |
| `telefone` | texto | ✅ | Formato válido |
| `criado_em` | data/hora | gerado auto | — |

**categorias**

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `nome` | texto | ✅ | Único no banco |

**produtos**

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `nome` | texto | ✅ | Único no banco |
| `descricao` | texto | ❌ | — |
| `preco` | decimal | ✅ | Maior que zero |
| `estoque` | inteiro | ✅ | Não pode ser negativo |
| `categoria_id` | inteiro | ✅ | Deve existir no banco |
| `ativo` | booleano | — | Padrão `true` |

**pedidos**

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `cliente_id` | inteiro | ✅ | Deve existir no banco |
| `status` | enum | — | Padrão `PENDENTE` |
| `valor_total` | decimal | gerado auto | Calculado a partir dos itens |
| `criado_em` | data/hora | gerado auto | — |
| `atualizado_em` | data/hora | gerado auto | — |

**itens_pedido**

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `pedido_id` | inteiro | gerado auto | — |
| `produto_id` | inteiro | ✅ | Deve existir e estar ativo |
| `quantidade` | inteiro | ✅ | Maior que zero |
| `preco_unitario` | decimal | gerado auto | Copiado do produto no momento do pedido |

> **Importante:** O `preco_unitario` deve ser copiado do produto no momento da criação do pedido — não referencie o preço atual do produto, pois ele pode mudar.

> **`valor_total` do pedido:** Deve ser calculado automaticamente como a soma de `quantidade × preco_unitario` de todos os itens. Sempre que os itens mudarem, o `valor_total` deve ser recalculado.

### 🔗 Relacionamentos

Um resumo das cardinalidades para guiar suas entidades JPA e os `JOIN`s dos relatórios:

| Relação | Cardinalidade | Lado dono da FK |
|---|---|---|
| Cliente → Pedidos | 1 : N | `pedidos.cliente_id` |
| Categoria → Produtos | 1 : N | `produtos.categoria_id` |
| Pedido → Itens | 1 : N | `itens_pedido.pedido_id` |
| Produto → Itens de pedido | 1 : N | `itens_pedido.produto_id` |

```
                ┌────────────┐
                │  categorias│
                └─────┬──────┘
                      │ 1:N
                ┌─────▼──────┐        ┌────────────┐
                │  produtos  │◄───────┤itens_pedido│
                └────────────┘  N:1   └─────┬──────┘
                                            │ N:1
┌────────────┐        ┌────────────┐        │
│  clientes  │───────►│   pedidos  │◄───────┘
└────────────┘  1:N   └────────────┘  1:N
```

---

## 🔄 Status do Pedido

O pedido segue um fluxo de status com transições válidas. Transições inválidas devem retornar `400` com mensagem clara.

```
PENDENTE ──→ CONFIRMADO ──→ EM_PREPARO ──→ ENVIADO ──→ ENTREGUE
    │               │
    └───────────────┴──→ CANCELADO
```

**Regras:**
- `PENDENTE` pode ir para `CONFIRMADO` ou `CANCELADO`
- `CONFIRMADO` pode ir para `EM_PREPARO` ou `CANCELADO`
- `EM_PREPARO` pode ir para `ENVIADO` apenas
- `ENVIADO` pode ir para `ENTREGUE` apenas
- `ENTREGUE` e `CANCELADO` são estados finais — nenhuma transição é permitida

---

## 🔌 Endpoints

Cada grupo abaixo traz a tabela de rotas seguida dos **modelos de requisição e resposta**. Os dados dos exemplos são fictícios e ilustram apenas o **formato** esperado.

---

### 👤 Clientes

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| `POST` | `/clientes` | Cadastra um cliente | `201` | `400` / `409` |
| `GET` | `/clientes` | Lista todos | `200` | — |
| `GET` | `/clientes/{id}` | Busca por ID | `200` | `404` |
| `PUT` | `/clientes/{id}` | Atualiza | `200` | `400` / `404` |
| `DELETE` | `/clientes/{id}` | Remove | `204` | `404` / `400` |

> Não é permitido remover um cliente que possui pedidos. Retorne `400` com mensagem explicativa.

#### `POST /clientes`

**Requisição**
```json
{
  "nome": "Ana Lima",
  "email": "ana.lima@email.com",
  "telefone": "(11) 98765-4321"
}
```

**Resposta `201`**
```json
{
  "id": 1,
  "nome": "Ana Lima",
  "email": "ana.lima@email.com",
  "telefone": "(11) 98765-4321",
  "criado_em": "2026-06-01T09:30:00"
}
```

**Erro `409` — e-mail já cadastrado**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 409,
  "mensagem": "Já existe um cliente cadastrado com o e-mail 'ana.lima@email.com'."
}
```

**Erro `400` — validação**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 400,
  "mensagem": "Erro de validação em campos",
  "erros": [
    { "campo": "nome", "mensagem": "O nome deve ter no mínimo 2 caracteres." },
    { "campo": "telefone", "mensagem": "O telefone é obrigatório." }
  ]
}
```

#### `GET /clientes`

**Resposta `200`**
```json
[
  {
    "id": 1,
    "nome": "Ana Lima",
    "email": "ana.lima@email.com",
    "telefone": "(11) 98765-4321",
    "criado_em": "2026-06-01T09:30:00"
  },
  {
    "id": 2,
    "nome": "Carlos Souza",
    "email": "carlos.souza@email.com",
    "telefone": "(21) 99876-5432",
    "criado_em": "2026-06-01T09:45:00"
  }
]
```

> Lista vazia retorna `200` com `[]`.

#### `GET /clientes/{id}`

**Resposta `200`**
```json
{
  "id": 1,
  "nome": "Ana Lima",
  "email": "ana.lima@email.com",
  "telefone": "(11) 98765-4321",
  "criado_em": "2026-06-01T09:30:00"
}
```

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 404,
  "mensagem": "Cliente com id 99 não encontrado."
}
```

#### `PUT /clientes/{id}`

**Requisição**
```json
{
  "nome": "Ana Lima Pereira",
  "email": "ana.pereira@email.com",
  "telefone": "(11) 98765-0000"
}
```

**Resposta `200`**
```json
{
  "id": 1,
  "nome": "Ana Lima Pereira",
  "email": "ana.pereira@email.com",
  "telefone": "(11) 98765-0000",
  "criado_em": "2026-06-01T09:30:00"
}
```

> Erros possíveis: `404` (cliente não existe) e `400` (validação / e-mail duplicado).

#### `DELETE /clientes/{id}`

**Resposta `204`** — sem corpo.

**Erro `400` — cliente possui pedidos**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 400,
  "mensagem": "Não é possível remover o cliente 'Ana Lima' pois ele possui 3 pedido(s) registrado(s)."
}
```

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 404,
  "mensagem": "Cliente com id 99 não encontrado."
}
```

---

### 🏷️ Categorias

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| `POST` | `/categorias` | Cadastra uma categoria | `201` | `400` / `409` |
| `GET` | `/categorias` | Lista todas | `200` | — |
| `DELETE` | `/categorias/{id}` | Remove | `204` | `404` / `400` |

> Não é permitido remover uma categoria que possui produtos vinculados.

#### `POST /categorias`

**Requisição**
```json
{ "nome": "Roupas" }
```

**Resposta `201`**
```json
{ "id": 1, "nome": "Roupas" }
```

**Erro `409` — nome já existe**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 409,
  "mensagem": "Já existe uma categoria com o nome 'Roupas'."
}
```

#### `GET /categorias`

**Resposta `200`**
```json
[
  { "id": 1, "nome": "Roupas" },
  { "id": 2, "nome": "Acessórios" },
  { "id": 3, "nome": "Calçados" },
  { "id": 4, "nome": "Eletrônicos" },
  { "id": 5, "nome": "Casa" }
]
```

#### `DELETE /categorias/{id}`

**Resposta `204`** — sem corpo.

**Erro `400` — categoria com produtos vinculados**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 400,
  "mensagem": "Não é possível remover a categoria 'Roupas' pois existem 8 produto(s) vinculado(s)."
}
```

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 404,
  "mensagem": "Categoria com id 99 não encontrada."
}
```

---

### 📦 Produtos

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| `POST` | `/produtos` | Cadastra um produto | `201` | `400` |
| `GET` | `/produtos` | Lista todos os ativos | `200` | — |
| `GET` | `/produtos/{id}` | Busca por ID | `200` | `404` |
| `PUT` | `/produtos/{id}` | Atualiza | `200` | `400` / `404` |
| `DELETE` | `/produtos/{id}` | Inativa o produto | `204` | `404` |

> O `DELETE` não remove o produto do banco — apenas marca como `ativo = false`. Produtos inativos não aparecem na listagem e não podem ser adicionados a novos pedidos.

#### `POST /produtos`

**Requisição**
```json
{
  "nome": "Camiseta Azul",
  "descricao": "Camiseta 100% algodão, gola redonda",
  "preco": 89.90,
  "estoque": 50,
  "categoria_id": 1
}
```

**Resposta `201`**
```json
{
  "id": 3,
  "nome": "Camiseta Azul",
  "descricao": "Camiseta 100% algodão, gola redonda",
  "preco": 89.90,
  "estoque": 50,
  "categoria": { "id": 1, "nome": "Roupas" },
  "ativo": true
}
```

**Erro `400` — categoria inexistente**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 400,
  "mensagem": "A categoria informada (id: 99) não existe."
}
```

**Erro `400` — validação**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 400,
  "mensagem": "Erro de validação em campos",
  "erros": [
    { "campo": "preco", "mensagem": "O preço deve ser maior que zero." },
    { "campo": "estoque", "mensagem": "O estoque não pode ser negativo." }
  ]
}
```

#### `GET /produtos`

**Resposta `200`**
```json
[
  {
    "id": 3,
    "nome": "Camiseta Azul",
    "descricao": "Camiseta 100% algodão, gola redonda",
    "preco": 89.90,
    "estoque": 50,
    "categoria": { "id": 1, "nome": "Roupas" },
    "ativo": true
  },
  {
    "id": 7,
    "nome": "Boné Preto",
    "descricao": null,
    "preco": 119.90,
    "estoque": 30,
    "categoria": { "id": 2, "nome": "Acessórios" },
    "ativo": true
  }
]
```

> Produtos com `ativo = false` **não aparecem** nesta listagem.

#### `GET /produtos/{id}`

**Resposta `200`**
```json
{
  "id": 3,
  "nome": "Camiseta Azul",
  "descricao": "Camiseta 100% algodão, gola redonda",
  "preco": 89.90,
  "estoque": 50,
  "categoria": { "id": 1, "nome": "Roupas" },
  "ativo": true
}
```

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 404,
  "mensagem": "Produto com id 99 não encontrado."
}
```

#### `PUT /produtos/{id}`

**Requisição**
```json
{
  "nome": "Camiseta Azul Marinho",
  "descricao": "Camiseta 100% algodão, gola redonda",
  "preco": 94.90,
  "estoque": 45,
  "categoria_id": 1
}
```

**Resposta `200`**
```json
{
  "id": 3,
  "nome": "Camiseta Azul Marinho",
  "descricao": "Camiseta 100% algodão, gola redonda",
  "preco": 94.90,
  "estoque": 45,
  "categoria": { "id": 1, "nome": "Roupas" },
  "ativo": true
}
```

> ⚠️ Alterar o `preco` **não muda** o `preco_unitario` de pedidos já criados — eles guardam o valor congelado no momento da compra.

#### `DELETE /produtos/{id}`

**Resposta `204`** — sem corpo. O produto **não é removido**; apenas recebe `ativo = false`.

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T09:30:00",
  "status": 404,
  "mensagem": "Produto com id 99 não encontrado."
}
```

---

### 🛒 Pedidos

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| `POST` | `/pedidos` | Cria um pedido com itens | `201` | `400` / `404` |
| `GET` | `/pedidos` | Lista todos | `200` | — |
| `GET` | `/pedidos/{id}` | Busca por ID com itens | `200` | `404` |
| `PATCH` | `/pedidos/{id}/status` | Avança o status do pedido | `200` | `400` / `404` |
| `DELETE` | `/pedidos/{id}` | Cancela o pedido | `204` | `400` / `404` |

> O `DELETE` não remove o pedido do banco — muda o status para `CANCELADO`, se a transição for permitida.

**Filtros disponíveis em `GET /pedidos`:**

```
?status=PENDENTE
?cliente_id=1
?status=CONFIRMADO&cliente_id=2
```

> Lista vazia não é erro — retorne `200` com `[]`.

> 💡 A criação do pedido deve ser **atômica**: validar cliente, validar cada item (existência, ativo e estoque), copiar o `preco_unitario`, dar baixa no estoque e calcular o `valor_total` devem acontecer na mesma transação. Se qualquer item falhar, nada é persistido.

#### `POST /pedidos`

**Requisição**
```bash
curl -X POST http://localhost:8080/pedidos \
  -H "Content-Type: application/json" \
  -d '{
    "cliente_id": 1,
    "itens": [
      { "produto_id": 3, "quantidade": 2 },
      { "produto_id": 7, "quantidade": 1 }
    ]
  }'
```

**Resposta `201`**
```json
{
  "id": 1,
  "cliente": { "id": 1, "nome": "Ana Lima" },
  "status": "PENDENTE",
  "valor_total": 299.70,
  "itens": [
    { "produto_id": 3, "nome": "Camiseta Azul", "quantidade": 2, "preco_unitario": 89.90, "subtotal": 179.80 },
    { "produto_id": 7, "nome": "Boné Preto", "quantidade": 1, "preco_unitario": 119.90, "subtotal": 119.90 }
  ],
  "criado_em": "2026-06-01T10:00:00",
  "atualizado_em": "2026-06-01T10:00:00"
}
```

**Erro `404` — cliente inexistente**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 404,
  "mensagem": "Cliente com id 99 não encontrado."
}
```

**Erro `400` — produto inativo**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "O produto 'Camiseta Azul' está inativo e não pode ser adicionado ao pedido."
}
```

**Erro `400` — estoque insuficiente**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "Estoque insuficiente para o produto 'Boné Preto'. Disponível: 3, solicitado: 5."
}
```

**Erro `400` — pedido sem itens**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "Erro de validação em campos",
  "erros": [
    { "campo": "itens", "mensagem": "O pedido deve ter pelo menos 1 item." }
  ]
}
```

#### `GET /pedidos`

Versão resumida — os itens aparecem apenas na busca por ID.

**Resposta `200`**
```json
[
  {
    "id": 1,
    "cliente": { "id": 1, "nome": "Ana Lima" },
    "status": "PENDENTE",
    "valor_total": 299.70,
    "criado_em": "2026-06-01T10:00:00"
  },
  {
    "id": 2,
    "cliente": { "id": 2, "nome": "Carlos Souza" },
    "status": "ENTREGUE",
    "valor_total": 540.00,
    "criado_em": "2026-05-28T14:20:00"
  }
]
```

**Com filtros** — `GET /pedidos?status=PENDENTE&cliente_id=1`
```json
[
  {
    "id": 1,
    "cliente": { "id": 1, "nome": "Ana Lima" },
    "status": "PENDENTE",
    "valor_total": 299.70,
    "criado_em": "2026-06-01T10:00:00"
  }
]
```

#### `GET /pedidos/{id}`

**Resposta `200`**
```json
{
  "id": 1,
  "cliente": { "id": 1, "nome": "Ana Lima" },
  "status": "CONFIRMADO",
  "valor_total": 299.70,
  "itens": [
    { "produto_id": 3, "nome": "Camiseta Azul", "quantidade": 2, "preco_unitario": 89.90, "subtotal": 179.80 },
    { "produto_id": 7, "nome": "Boné Preto", "quantidade": 1, "preco_unitario": 119.90, "subtotal": 119.90 }
  ],
  "criado_em": "2026-06-01T10:00:00",
  "atualizado_em": "2026-06-01T10:15:00"
}
```

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 404,
  "mensagem": "Pedido com id 99 não encontrado."
}
```

#### `PATCH /pedidos/{id}/status`

**Requisição**
```bash
curl -X PATCH http://localhost:8080/pedidos/1/status \
  -H "Content-Type: application/json" \
  -d '{ "status": "CONFIRMADO" }'
```

**Resposta `200`**
```json
{
  "id": 1,
  "status": "CONFIRMADO",
  "atualizado_em": "2026-06-01T10:15:00"
}
```

**Erro `400` — transição inválida**
```json
{
  "timestamp": "2026-06-01T10:15:00",
  "status": 400,
  "mensagem": "Transição inválida: pedido EM_PREPARO só pode ir para ENVIADO."
}
```

**Erro `400` — estado final**
```json
{
  "timestamp": "2026-06-01T10:15:00",
  "status": 400,
  "mensagem": "Transição inválida: pedido ENTREGUE não pode mudar de status."
}
```

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T10:15:00",
  "status": 404,
  "mensagem": "Pedido com id 99 não encontrado."
}
```

#### `DELETE /pedidos/{id}`

Cancela o pedido — muda o status para `CANCELADO`, se a transição for permitida.

**Resposta `204`** — sem corpo.

**Erro `400` — não pode cancelar**
```json
{
  "timestamp": "2026-06-01T10:15:00",
  "status": 400,
  "mensagem": "Transição inválida: pedido ENTREGUE não pode ser cancelado."
}
```

**Erro `404`**
```json
{
  "timestamp": "2026-06-01T10:15:00",
  "status": 404,
  "mensagem": "Pedido com id 99 não encontrado."
}
```

---

### 📊 Relatórios

Os endpoints de relatório aceitam os query params `data_inicio` e `data_fim` (formato `YYYY-MM-DD`). **Ambos são obrigatórios.**

| Método | Rota | Descrição | Sucesso |
|---|---|---|---|
| `GET` | `/relatorios/pedidos/resumo` | Total de pedidos e receita por status no período | `200` |
| `GET` | `/relatorios/pedidos/por-dia` | Receita e quantidade de pedidos por dia | `200` |
| `GET` | `/relatorios/produtos/mais-vendidos` | Ranking de produtos por quantidade e receita | `200` |
| `GET` | `/relatorios/categorias/receita` | Receita total por categoria no período | `200` |
| `GET` | `/relatorios/clientes/ticket-medio` | Ticket médio por cliente no período | `200` |

> Se faltar `data_inicio` ou `data_fim`, retorne `400`:
> ```json
> {
>   "timestamp": "2026-06-01T10:00:00",
>   "status": 400,
>   "mensagem": "Os parâmetros 'data_inicio' e 'data_fim' são obrigatórios."
> }
> ```

#### `GET /relatorios/pedidos/resumo`

```bash
curl "http://localhost:8080/relatorios/pedidos/resumo?data_inicio=2026-01-01&data_fim=2026-01-31"
```

**Resposta `200`**
```json
{
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-31" },
  "resumo": [
    { "status": "ENTREGUE", "quantidade": 312, "receita": 48750.00 },
    { "status": "CANCELADO", "quantidade": 27, "receita": 0.00 },
    { "status": "PENDENTE", "quantidade": 14, "receita": 2100.00 }
  ],
  "total_pedidos": 353,
  "receita_total": 50850.00,
  "taxa_cancelamento": "7.65%"
}
```

#### `GET /relatorios/pedidos/por-dia`

```bash
curl "http://localhost:8080/relatorios/pedidos/por-dia?data_inicio=2026-01-01&data_fim=2026-01-31"
```

**Resposta `200`**
```json
{
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-31" },
  "dias": [
    { "data": "2026-01-01", "quantidade_pedidos": 12, "receita": 1850.00 },
    { "data": "2026-01-02", "quantidade_pedidos": 8, "receita": 1320.50 },
    { "data": "2026-01-03", "quantidade_pedidos": 15, "receita": 2410.00 }
  ],
  "total_pedidos": 353,
  "receita_total": 50850.00
}
```

> Ordene os dias em ordem crescente de data. Dias sem pedidos podem ser omitidos (mais simples) ou preenchidos com zero (mais completo) — documente sua escolha.

#### `GET /relatorios/produtos/mais-vendidos`

```bash
curl "http://localhost:8080/relatorios/produtos/mais-vendidos?data_inicio=2026-01-01&data_fim=2026-01-31"
```

**Resposta `200`**
```json
{
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-31" },
  "produtos": [
    { "posicao": 1, "id": 3, "nome": "Camiseta Azul", "categoria": "Roupas", "quantidade_vendida": 97, "receita": 8720.30 },
    { "posicao": 2, "id": 7, "nome": "Boné Preto", "categoria": "Acessórios", "quantidade_vendida": 84, "receita": 10071.60 }
  ]
}
```

#### `GET /relatorios/categorias/receita`

```bash
curl "http://localhost:8080/relatorios/categorias/receita?data_inicio=2026-01-01&data_fim=2026-01-31"
```

**Resposta `200`**
```json
{
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-31" },
  "categorias": [
    { "posicao": 1, "id": 1, "nome": "Roupas", "quantidade_vendida": 412, "receita": 28750.00 },
    { "posicao": 2, "id": 2, "nome": "Acessórios", "quantidade_vendida": 198, "receita": 15300.00 },
    { "posicao": 3, "id": 3, "nome": "Calçados", "quantidade_vendida": 95, "receita": 6800.00 }
  ],
  "receita_total": 50850.00
}
```

#### `GET /relatorios/clientes/ticket-medio`

```bash
curl "http://localhost:8080/relatorios/clientes/ticket-medio?data_inicio=2026-01-01&data_fim=2026-01-31"
```

**Resposta `200`**
```json
{
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-31" },
  "clientes": [
    { "posicao": 1, "id": 2, "nome": "Carlos Souza", "total_pedidos": 8, "receita_total": 4200.00, "ticket_medio": 525.00 },
    { "posicao": 2, "id": 1, "nome": "Ana Lima", "total_pedidos": 12, "receita_total": 5100.00, "ticket_medio": 425.00 }
  ]
}
```

> `ticket_medio = receita_total / total_pedidos`. O esperado é **não** contar pedidos `CANCELADO` na receita.

---

## ❌ Catálogo de erros

Use este resumo para padronizar as mensagens e cobrir os casos nos testes.

| Situação | Status | Mensagem (exemplo) |
|---|---|---|
| Campo inválido no corpo | `400` | `Erro de validação em campos` (+ array `erros`) |
| Transição de status inválida | `400` | `Transição inválida: pedido X não pode mudar de status.` |
| Produto inativo no pedido | `400` | `O produto 'X' está inativo e não pode ser adicionado ao pedido.` |
| Estoque insuficiente | `400` | `Estoque insuficiente para o produto 'X'. Disponível: N, solicitado: M.` |
| Remover cliente com pedidos | `400` | `Não é possível remover o cliente 'X' pois ele possui N pedido(s)...` |
| Remover categoria com produtos | `400` | `Não é possível remover a categoria 'X' pois existem N produto(s)...` |
| Parâmetros de relatório ausentes | `400` | `Os parâmetros 'data_inicio' e 'data_fim' são obrigatórios.` |
| Recurso não encontrado | `404` | `X com id N não encontrado.` |
| E-mail / nome duplicado | `409` | `Já existe um(a) X com ...` |

### Exemplos detalhados

**Produto inativo no pedido — `400`**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "O produto 'Camiseta Azul' está inativo e não pode ser adicionado ao pedido."
}
```

**Estoque insuficiente — `400`**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "Estoque insuficiente para o produto 'Boné Preto'. Disponível: 3, solicitado: 5."
}
```

**Erro de validação em campos — `400`**
```json
{
  "timestamp": "2026-06-01T10:00:00",
  "status": 400,
  "mensagem": "Erro de validação em campos",
  "erros": [
    { "campo": "itens", "mensagem": "O pedido deve ter pelo menos 1 item." }
  ]
}
```

---

## 🏗️ Organização do código

```
controller  →  recebe a requisição e devolve a resposta HTTP
service     →  aplica as regras de negócio e as transições de status
repository  →  executa as queries no banco
model       →  entidades JPA com os relacionamentos
dto         →  objetos de entrada (request) e saída (response)
projection  →  interfaces de projeção para queries nativas com agregação
mapper      →  conversão entre entidade e DTO
exception   →  exceções customizadas e handler global
```

> Use as funções do banco (`SUM`, `COUNT`, `GROUP BY`, `JOIN`) nos relatórios — não traga todos os registros para a memória e processe em Java.

---

## ⭐ Bônus

Terminou tudo? Tente implementar também:

- **Filtro de pedidos por status** — `GET /pedidos?status=PENDENTE`
- **Filtro de produtos por categoria** — `GET /produtos?categoria_id=1`
- **Paginação nas listagens** — `GET /pedidos?page=0&size=20`
- **Restaurar estoque ao cancelar** — ao cancelar um pedido, devolva a quantidade dos itens ao estoque de cada produto

### Modelos de resposta dos bônus

**Filtro de produtos por categoria** — `GET /produtos?categoria_id=1`
```json
[
  {
    "id": 3,
    "nome": "Camiseta Azul",
    "preco": 89.90,
    "estoque": 50,
    "categoria": { "id": 1, "nome": "Roupas" },
    "ativo": true
  }
]
```

**Paginação (estilo `Page` do Spring)** — `GET /pedidos?page=0&size=20`
```json
{
  "content": [
    {
      "id": 1,
      "cliente": { "id": 1, "nome": "Ana Lima" },
      "status": "PENDENTE",
      "valor_total": 299.70,
      "criado_em": "2026-06-01T10:00:00"
    }
  ],
  "page": 0,
  "size": 20,
  "total_elements": 142,
  "total_pages": 8,
  "first": true,
  "last": false
}
```

---

## 🌱 Script de seed

Um arquivo `import.sql` está disponível na raiz do repositório. Ele popula o banco com:

- 5 categorias
- 15 produtos distribuídos entre as categorias
- 10 clientes
- Pedidos em diferentes status distribuídos em 3 meses

Para executar:

```bash
psql -U seu_usuario -d seu_banco -f import.sql
```

> Os dados são distribuídos de forma irregular entre clientes, produtos e dias — para que os filtros de data e os relatórios façam diferença nos resultados.

---

## 🚀 Como entregar

Consulte o [CONTRIBUTING.md](../../CONTRIBUTING.md) para ver como compartilhar sua solução.

---

## 💡 Dicas

- **Comece pelo modelo de dados** — desenhe as relações no papel antes de abrir o IDE
- **Implemente o CRUD antes dos relatórios** — você vai precisar das entidades criadas e populadas para testar
- **O `valor_total` do pedido é calculado** — some `quantidade × preco_unitario` de cada item no momento da criação
- **O `preco_unitario` do item é fixo** — copie o preço do produto ao criar o item, não guarde referência
- **Valide o estoque antes de criar o pedido** — verifique se tem unidades disponíveis e atualize após confirmar
- **Centralize o tratamento de erro** — um `@ControllerAdvice` único mantém todas as respostas no formato padrão
- **Testes automatizados e Docker são diferenciais**, não requisitos

Boa sorte! 💙
