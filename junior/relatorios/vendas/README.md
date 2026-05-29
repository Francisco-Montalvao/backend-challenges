# 🛒 Relatório de Vendas

![Nível](https://img.shields.io/badge/nível-júnior-4CAF50?style=flat-square)
![Tema](https://img.shields.io/badge/tema-CRUD%20%2B%20Agregações-2196F3?style=flat-square)
![Tempo](https://img.shields.io/badge/tempo-3%20a%204%20dias-FF9800?style=flat-square)

CRUD é só o começo. Neste desafio você vai cadastrar entidades, aplicar validações e depois trabalhar com agregações — fazer o banco de dados calcular totais e rankings, em vez de trazer tudo para a memória. É uma habilidade essencial para qualquer desenvolvedor backend.

---

## 📋 O que você vai construir

Um pequeno PDV (Ponto de Venda) registra vendas diariamente. Os donos precisam de um dashboard com relatórios consolidados. Você vai construir o cadastro de vendedores e vendas, e os endpoints que alimentam esses gráficos.

---

## 🗄️ Modelo de Dados

Monte o banco com estas tabelas e popule com dados de teste (seed):

**vendedores**

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | inteiro | PK |
| `nome` | texto | Nome do vendedor |
| `email` | texto | E-mail do vendedor |
| `telefone` | texto | Telefone do vendedor |

**vendas**

| Campo | Tipo | Descrição |
|---|---|---|
| `id` | inteiro | PK |
| `vendedor_id` | inteiro | FK para vendedores |
| `valor_total` | decimal | Valor da venda |
| `data` | data | Data da venda (YYYY-MM-DD) |

> Você define a estrutura exata — o que importa é conseguir responder as queries dos endpoints.

---

## 🔌 Endpoints

### Geral

| Método | Rota | Descrição | Sucesso |
|---|---|---|---|
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/health` | Verifica se a API está no ar | 200 |

### Vendedores

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| ![POST](https://img.shields.io/badge/POST-49CC90?style=flat-square) | `/vendedores` | Cadastra um vendedor | 201 | 400 |
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/vendedores` | Lista todos os vendedores | 200 | — |
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/vendedores/:id` | Busca um vendedor pelo id | 200 | 404 |
| ![PUT](https://img.shields.io/badge/PUT-FCA130?style=flat-square) | `/vendedores/:id` | Atualiza um vendedor | 200 | 400, 404 |
| ![DELETE](https://img.shields.io/badge/DELETE-F93E3E?style=flat-square) | `/vendedores/:id` | Remove um vendedor | 204 | 404 |

### Vendas

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| ![POST](https://img.shields.io/badge/POST-49CC90?style=flat-square) | `/vendas` | Cadastra uma venda | 201 | 400 |
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/vendas` | Lista todas as vendas | 200 | — |
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/vendas/:id` | Busca uma venda pelo id | 200 | 404 |
| ![PUT](https://img.shields.io/badge/PUT-FCA130?style=flat-square) | `/vendas/:id` | Atualiza uma venda | 200 | 400, 404 |
| ![DELETE](https://img.shields.io/badge/DELETE-F93E3E?style=flat-square) | `/vendas/:id` | Remove uma venda | 204 | 404 |

### Relatórios

Os endpoints de relatório aceitam os query params `data_inicio` e `data_fim` (formato `YYYY-MM-DD`). **Ambos são obrigatórios.**

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/relatorios/vendas/total` | Total vendido e quantidade no período | 200 | 400 |
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/relatorios/vendas/por-vendedor` | Ranking de vendedores no período | 200 | 400 |
| ![GET](https://img.shields.io/badge/GET-61AFFE?style=flat-square) | `/relatorios/vendas/diario` | Total agrupado por dia no período | 200 | 400 |

---

## ✅ Validações

### Vendedores

| Campo | Regra |
|---|---|
| `nome` | Obrigatório, mínimo 2 caracteres |
| `email` | Obrigatório, formato válido, único no banco |
| `telefone` | Obrigatório, mínimo 8 caracteres |

### Vendas

| Campo | Regra |
|---|---|
| `vendedor_id` | Obrigatório, deve existir na tabela `vendedores` |
| `valor_total` | Obrigatório, maior que zero |
| `data` | Obrigatório, formato YYYY-MM-DD, não pode ser data futura |

---

## 💡 Exemplos de requisição e resposta

### Health check

```bash
curl "http://localhost:8080/health"
```
```json
// 200 OK
{ "status": "ok" }
```

---

### Vendedores

<details>
<summary><strong>POST /vendedores — Cadastrar vendedor</strong></summary>

```bash
curl -X POST "http://localhost:8080/vendedores" \
  -H "Content-Type: application/json" \
  -d '{ "nome": "Carla Souza", "email": "carla@pdv.com", "telefone": "11999990001" }'
```

```json
// 201 Created
{ "id": 1, "nome": "Carla Souza", "email": "carla@pdv.com", "telefone": "11999990001" }
```

```json
// 400 Bad Request
{ "erro": "email já está em uso." }
```
</details>

<details>
<summary><strong>GET /vendedores — Listar vendedores</strong></summary>

```bash
curl "http://localhost:8080/vendedores"
```

```json
// 200 OK
[
  { "id": 1, "nome": "Carla Souza", "email": "carla@pdv.com", "telefone": "11999990001" },
  { "id": 2, "nome": "João Silva", "email": "joao@pdv.com", "telefone": "11999990002" }
]
```
</details>

<details>
<summary><strong>GET /vendedores/:id — Buscar por id</strong></summary>

```bash
curl "http://localhost:8080/vendedores/1"
```

```json
// 200 OK
{ "id": 1, "nome": "Carla Souza", "email": "carla@pdv.com", "telefone": "11999990001" }
```

```json
// 404 Not Found
{ "erro": "Vendedor não encontrado." }
```
</details>

<details>
<summary><strong>PUT /vendedores/:id — Atualizar vendedor</strong></summary>

```bash
curl -X PUT "http://localhost:8080/vendedores/1" \
  -H "Content-Type: application/json" \
  -d '{ "telefone": "11988880001" }'
```

```json
// 200 OK
{ "id": 1, "nome": "Carla Souza", "email": "carla@pdv.com", "telefone": "11988880001" }
```

```json
// 404 Not Found
{ "erro": "Vendedor não encontrado." }
```
</details>

<details>
<summary><strong>DELETE /vendedores/:id — Remover vendedor</strong></summary>

```bash
curl -X DELETE "http://localhost:8080/vendedores/1"
```

```
// 204 No Content
```

```json
// 404 Not Found
{ "erro": "Vendedor não encontrado." }
```
</details>

---

### Vendas

<details>
<summary><strong>POST /vendas — Cadastrar venda</strong></summary>

```bash
curl -X POST "http://localhost:8080/vendas" \
  -H "Content-Type: application/json" \
  -d '{ "vendedor_id": 1, "valor_total": 350.00, "data": "2026-01-15" }'
```

```json
// 201 Created
{ "id": 1, "vendedor_id": 1, "valor_total": 350.00, "data": "2026-01-15" }
```

```json
// 400 Bad Request
{ "erro": "vendedor_id não encontrado." }
```

```json
// 400 Bad Request
{ "erro": "valor_total deve ser maior que zero." }
```

```json
// 400 Bad Request
{ "erro": "data não pode ser uma data futura." }
```
</details>

<details>
<summary><strong>GET /vendas — Listar vendas</strong></summary>

```bash
curl "http://localhost:8080/vendas"
```

```json
// 200 OK
[
  { "id": 1, "vendedor_id": 1, "valor_total": 350.00, "data": "2026-01-15" },
  { "id": 2, "vendedor_id": 2, "valor_total": 780.50, "data": "2026-01-15" }
]
```
</details>

<details>
<summary><strong>GET /vendas/:id — Buscar por id</strong></summary>

```bash
curl "http://localhost:8080/vendas/1"
```

```json
// 200 OK
{ "id": 1, "vendedor_id": 1, "valor_total": 350.00, "data": "2026-01-15" }
```

```json
// 404 Not Found
{ "erro": "Venda não encontrada." }
```
</details>

<details>
<summary><strong>PUT /vendas/:id — Atualizar venda</strong></summary>

```bash
curl -X PUT "http://localhost:8080/vendas/1" \
  -H "Content-Type: application/json" \
  -d '{ "valor_total": 400.00 }'
```

```json
// 200 OK
{ "id": 1, "vendedor_id": 1, "valor_total": 400.00, "data": "2026-01-15" }
```

```json
// 404 Not Found
{ "erro": "Venda não encontrada." }
```
</details>

<details>
<summary><strong>DELETE /vendas/:id — Remover venda</strong></summary>

```bash
curl -X DELETE "http://localhost:8080/vendas/1"
```

```
// 204 No Content
```

```json
// 404 Not Found
{ "erro": "Venda não encontrada." }
```
</details>

---

### Relatórios

<details>
<summary><strong>GET /relatorios/vendas/total — Total no período</strong></summary>

```bash
curl "http://localhost:8080/relatorios/vendas/total?data_inicio=2026-01-01&data_fim=2026-01-31"
```

```json
// 200 OK
{
  "total_vendido": 48750.00,
  "quantidade_vendas": 312,
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-31" }
}
```

> Se não houver vendas no período, retorne os campos zerados — não retorne 404.

```json
// 200 OK — período sem vendas
{
  "total_vendido": 0.00,
  "quantidade_vendas": 0,
  "periodo": { "inicio": "2026-06-01", "fim": "2026-06-30" }
}
```

```json
// 400 Bad Request — parâmetro ausente
{ "erro": "data_fim é obrigatório. Use o formato YYYY-MM-DD." }
```

```json
// 400 Bad Request — formato inválido
{ "erro": "data_inicio está em formato inválido. Use YYYY-MM-DD." }
```

```json
// 400 Bad Request — intervalo invertido
{ "erro": "data_fim não pode ser anterior a data_inicio." }
```
</details>

<details>
<summary><strong>GET /relatorios/vendas/por-vendedor — Ranking no período</strong></summary>

```bash
curl "http://localhost:8080/relatorios/vendas/por-vendedor?data_inicio=2026-01-01&data_fim=2026-01-31"
```

> Ordene pelo `total_vendido` decrescente. Vendedores sem vendas no período não precisam aparecer.

```json
// 200 OK
{
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-31" },
  "vendedores": [
    { "posicao": 1, "id": 3, "nome": "Carla Souza", "total_vendido": 18200.00, "quantidade_vendas": 97 },
    { "posicao": 2, "id": 1, "nome": "João Silva", "total_vendido": 15400.00, "quantidade_vendas": 84 },
    { "posicao": 3, "id": 2, "nome": "Maria Oliveira", "total_vendido": 15150.00, "quantidade_vendas": 131 }
  ]
}
```
</details>

<details>
<summary><strong>GET /relatorios/vendas/diario — Total por dia no período</strong></summary>

```bash
curl "http://localhost:8080/relatorios/vendas/diario?data_inicio=2026-01-01&data_fim=2026-01-03"
```

> Retorne apenas os dias que tiveram ao menos uma venda — não precisa preencher dias vazios.

```json
// 200 OK
{
  "periodo": { "inicio": "2026-01-01", "fim": "2026-01-03" },
  "dias": [
    { "data": "2026-01-01", "total_vendido": 5200.00, "quantidade_vendas": 34 },
    { "data": "2026-01-02", "total_vendido": 4875.50, "quantidade_vendas": 29 },
    { "data": "2026-01-03", "total_vendido": 6100.00, "quantidade_vendas": 41 }
  ]
}
```
</details>

---

## 🏗️ Qualidade do código

Organize em camadas:

```
controller  →  valida os dados da requisição e responde HTTP
service     →  aplica as regras de negócio
repository  →  executa as queries no banco
```

> Use as funções do banco (`SUM`, `COUNT`, `GROUP BY`) nos relatórios — não traga todos os registros para a memória e some no código.

---

## 🌱 Script de seed

Popule o banco em duas etapas — primeiro os vendedores, depois as vendas (que dependem dos IDs já existentes):

**1. Cadastre os vendedores:**
- Mínimo de 5 vendedores com nome, e-mail e telefone realistas

**2. Cadastre as vendas:**
- Mínimo de 200 vendas distribuídas em pelo menos 3 meses diferentes
- Cada venda deve referenciar um `vendedor_id` existente
- Os valores de `valor_total` devem variar (não use o mesmo valor para todas)
- Distribua as vendas de forma irregular — alguns dias com mais, outros com menos

> Isso é importante para que os filtros de data façam diferença nos resultados.

---

## 🚀 Como entregar

1. Suba sua solução em um repositório público
2. Inclua no README o comando para rodar o seed e iniciar a API
3. Abra uma [Discussion](https://github.com/Francisco-Montalvao/backend-challenges/discussions) com o link

---

## 💬 Dicas finais

- Implemente o CRUD antes de partir para os relatórios — você vai precisar das entidades criadas para testar
- No DELETE de um vendedor, decida o que fazer com as vendas vinculadas a ele — delete em cascata ou bloqueie a remoção
- `data_inicio` e `data_fim` nos relatórios são obrigatórios — valide nesta ordem: ausente → formato inválido → intervalo invertido
- Query params chegam como string — converta para data antes de usar no banco
- O PostgreSQL pode retornar `valor_total` como string dependendo do driver — converta para número antes de responder
- Paginação nas listagens e exportação CSV são diferenciais opcionais

---

Boa sorte — CRUD sólido com validações é o que separa um júnior de um pleno nas entrevistas!
