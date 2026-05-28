# 💡 Por onde começar?

Travou em algum conceito? Sem problema — aqui estão os pontos onde a maioria das pessoas trava nesse desafio e como desbloquear cada um.

---

## 1. Relacionamento entre Produto e Categoria

Nesse desafio temos um relacionamento **Um para Muitos (1:N)**: uma categoria tem vários produtos, mas cada produto pertence a só uma categoria.

Na prática isso significa que a tabela `produtos` vai ter uma coluna `categoria_id` apontando para a tabela `categorias`.

Quando você buscar um produto, vai precisar trazer junto o nome da categoria — pesquise por **JOIN** (SQL) ou **populate/lookup** (NoSQL) na sua linguagem.

---

## 2. Status HTTP — qual usar em cada situação?

| Situação | Status |
|---|---|
| Criou com sucesso | `201 Created` |
| Buscou / atualizou com sucesso | `200 OK` |
| Deletou com sucesso | `204 No Content` |
| Dados inválidos ou regra violada | `400 Bad Request` |
| ID não encontrado | `404 Not Found` |
| Nome duplicado | `409 Conflict` |

Referência completa: [MDN — HTTP Status Codes](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)

---

## 3. Validação dos dados

Valide os dados **antes** de tentar salvar no banco. Se o preço vier negativo ou o `categoria_id` não existir, retorne o erro imediatamente — não deixe chegar no banco.

Procure pela biblioteca de validação da sua linguagem:

- **Node.js** → Zod ou Joi
- **Python** → Pydantic
- **Java** → Bean Validation
- **Go** → validator
- **Outras** → pesquise `"validation library [sua linguagem]"`

---

## 4. Filtros na URL (Query Parameters)

No `GET /produtos`, o candidato pode filtrar por categoria e faixa de preço usando a URL assim:

```
/produtos?categoria_id=1&preco_min=10&preco_max=100
```

Esses valores chegam como **query parameters** — pesquise como lê-los na sua framework. Lembre-se que eles chegam como string, então converta para número antes de usar.

---

## 5. Organização do código

Separe o código em camadas desde o início — fica muito mais fácil de corrigir e evoluir:

```
controller  →  recebe a requisição e devolve a resposta
service     →  valida e aplica as regras de negócio
repository  →  consulta e salva no banco
```

Se tudo estiver num arquivo só, tudo bem para começar — mas tente refatorar para camadas assim que funcionar.

---

## 🔗 Links úteis

- [MDN — HTTP Status Codes](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)
- [O que é modelagem de dados relacional](https://www.lucidchart.com/pages/pt/o-que-e-diagrama-entidade-relacionamento)
- [REST API — boas práticas](https://restfulapi.net/)
