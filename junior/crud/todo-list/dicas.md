# Por onde começar?

Travou? Totalmente normal. Este arquivo cobre os pontos onde a maioria das pessoas para.

## 1. Estrutura de pastas

Não precisa começar com a estrutura perfeita. Um arquivo só está ok para dar o pontapé:

```
index.js (ou main.go, app.py...)
  ├── POST /tarefas   →  valida e salva no banco
  ├── GET  /tarefas   →  busca todos os registros
  └── ...
```

Quando começar a repetir código, aí vale separar em controller → service → repository.

## 2. Status HTTP — qual usar em cada situação?

| Situação | Status |
|---|---|
| Criou com sucesso | 201 |
| Buscou / atualizou com sucesso | 200 |
| Deletou com sucesso (sem body) | 204 |
| Dados inválidos no request | 400 |
| ID não encontrado no banco | 404 |

Referência completa: [MDN — Códigos de status HTTP](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)

## 3. Validação dos dados

Valide ANTES de salvar no banco. Se o título veio vazio, retorne `400` imediatamente — não tente salvar e esperar o banco dar erro.

Libs por linguagem:
- **Node.js:** `zod` ou `joi`
- **Python:** `pydantic`
- **Java:** Bean Validation (`@NotNull`, `@Size`)
- **Go:** `go-playground/validator`

## 4. Como testar sem front-end

Use curl ou o [Insomnia](https://insomnia.rest/) para fazer requisições manualmente:

```bash
# Criar
curl -X POST http://localhost:8080/tarefas \
  -H "Content-Type: application/json" \
  -d '{"titulo": "Minha tarefa"}'

# Listar
curl http://localhost:8080/tarefas

# Deletar
curl -X DELETE http://localhost:8080/tarefas/1
```

## 5. Bônus: filtro por status

O endpoint `GET /tarefas?concluida=true` é um diferencial. Atenção: query params chegam como string — converta `"true"` para booleano antes de usar no banco.

---

Links úteis:
- [MDN — HTTP Status Codes (pt-BR)](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)
- [REST API boas práticas (restfulapi.net)](https://restfulapi.net/)
