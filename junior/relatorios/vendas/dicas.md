# Por onde começar?

Relatórios com agregações parecem complexos, mas ficam simples quando você entende o padrão. Este arquivo explica do zero.

## 1. Funções de agregação no banco

A regra de ouro: deixe o banco calcular, não o seu código.

```sql
-- Ruim: busca tudo e soma na aplicação
SELECT * FROM vendas WHERE data BETWEEN '2026-01-01' AND '2026-01-31';

-- Bom: o banco calcula e retorna só o resultado
SELECT SUM(valor_total), COUNT(*)
FROM vendas
WHERE data BETWEEN '2026-01-01' AND '2026-01-31';
```

Para o ranking de vendedores, use `GROUP BY` com `JOIN`:

```sql
SELECT v.nome, SUM(ve.valor_total) AS total, COUNT(*) AS quantidade
FROM vendas ve
JOIN vendedores v ON v.id = ve.vendedor_id
WHERE ve.data BETWEEN :inicio AND :fim
GROUP BY v.nome
ORDER BY total DESC;
```

## 2. Query params chegam como string

Os parâmetros `data_inicio` e `data_fim` chegam na rota como texto — valide o formato antes de usar na query:

```python
# Python
from datetime import datetime
try:
    data = datetime.strptime(data_inicio, "%Y-%m-%d").date()
except ValueError:
    return 400, {"erro": "data_inicio está em formato inválido. Use YYYY-MM-DD."}
```

Se `data_fim` vier antes de `data_inicio`, também retorne `400 Bad Request`.

## 3. Script de seed

Sem dados, não tem como testar. Crie um script que insira vendedores e vendas espalhadas por pelo menos 3 meses diferentes:

- 5 a 10 vendedores com nomes fictícios
- 100 a 300 vendas com valores e datas variadas

Ferramentas para gerar dados fictícios: Faker (Node.js/Python/Ruby), JavaFaker (Java), gofakeit (Go).

## 4. Status HTTP neste desafio

| Situação | Status |
|---|---|
| Relatório gerado com sucesso | 200 |
| Parâmetro ausente ou formato inválido | 400 |
| `data_fim` anterior a `data_inicio` | 400 |

## 5. Evite duplicar a validação de datas

Os três endpoints precisam validar `data_inicio` e `data_fim`. Crie uma função/método reutilizável para isso — você vai usar nos três lugares.

---

Links úteis:
- [SQL GROUP BY — W3Schools](https://www.w3schools.com/sql/sql_groupby.asp)
- [ISO 8601 — Formato de datas](https://pt.wikipedia.org/wiki/ISO_8601)
- [MDN — HTTP Status Codes (pt-BR)](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)
