# Relatório de Vendas

**Nível:** Júnior | **Tema:** Agregações | **Tempo estimado:** 2 a 3 dias

CRUD é só o começo. Neste desafio você vai trabalhar com agregações — fazer o banco de dados calcular totais e rankings, em vez de trazer tudo para a memória. É uma habilidade essencial para qualquer desenvolvedor backend.

## O que você vai construir

Um pequeno PDV (Ponto de Venda) registra vendas diariamente. Os donos precisam de um dashboard com relatórios consolidados. Você vai construir os endpoints que alimentam esses gráficos.

## Modelo de Dados

Monte o banco com pelo menos estas tabelas e popule com dados de teste (seed):

| Tabela | Campo | Tipo | Descrição |
|---|---|---|---|
| `vendedores` | `id` | inteiro | PK |
| `vendedores` | `nome` | texto | Nome do vendedor |
| `vendas` | `id` | inteiro | PK |
| `vendas` | `vendedor_id` | inteiro | FK para vendedores |
| `vendas` | `valor_total` | decimal | Valor da venda |
| `vendas` | `data` | data | Data da venda (YYYY-MM-DD) |

> Você define a estrutura exata — o que importa é conseguir responder as queries dos endpoints.

## Endpoints

Todos aceitam os query params `data_inicio` e `data_fim` (formato `YYYY-MM-DD`).

| Método | Rota | Descrição | Sucesso | Erro |
|---|---|---|---|---|
| GET | `/relatorios/vendas/total` | Total vendido e quantidade no período | 200 | 400 |
| GET | `/relatorios/vendas/por-vendedor` | Ranking de vendedores no período | 200 | 400 |
| GET | `/relatorios/vendas/diario` | Total agrupado por dia no período | 200 | 400 |

## Exemplos de requisição

**Total vendido em um período**

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

**Data em formato inválido**

```bash
curl "http://localhost:8080/relatorios/vendas/total?data_inicio=2026-13-45&data_fim=2026-01-31"
```

```json
// 400 Bad Request
{ "erro": "data_inicio está em formato inválido. Use YYYY-MM-DD." }
```

## Qualidade do código

Organize em camadas:

```
controller  →  valida query params e responde HTTP
service     →  monta os filtros e chama o repositório
repository  →  executa as queries com GROUP BY e SUM
```

> Use as funções do banco (`SUM`, `COUNT`, `GROUP BY`) — não traga todos os registros para a memória e some no código.

## Critérios de avaliação

| Critério | Peso |
|---|---|
| Endpoints retornando dados corretos com filtros de data | 40% |
| Agregações feitas no banco (não na memória) | 25% |
| Validação de parâmetros inválidos com 400 | 20% |
| Script de seed com dados realistas | 15% |

## Como entregar

1. Suba sua solução em um repositório público
2. Inclua no README o comando para rodar o seed e iniciar a API
3. Abra uma [Discussion](https://github.com/Francisco-Montalvao/backend-challenges/discussions) com o link

> Bônus: use o `docker-compose.yml` disponível na pasta para subir um PostgreSQL rapidamente com `docker compose up`.

## Dicas finais

- Valide se `data_fim` não é anterior a `data_inicio` — retorne `400`
- Query params chegam como string — converta para data antes de usar no banco
- Popule dados de seed em períodos diferentes para poder testar os filtros
- Paginação no ranking e exportação CSV são diferenciais opcionais

Boa sorte — queries de agregação aparecem em quase toda entrevista backend!
