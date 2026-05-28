# Login com JWT

**Nível:** Júnior | **Tema:** Autenticação | **Tempo estimado:** 3 a 4 dias

Autenticação é uma das habilidades mais requisitadas no mercado. Neste desafio você vai implementar um fluxo completo: cadastro, login e rotas protegidas com JWT.

## O que você vai construir

Uma aplicação precisa de um sistema de autenticação seguro. Os usuários devem conseguir se cadastrar, fazer login e acessar rotas que exigem token.

## Modelo de Dados

| Campo | Tipo | Obrigatório | Regra |
|---|---|---|---|
| `id` | inteiro | gerado auto | — |
| `nome` | texto | sim | — |
| `email` | texto | sim | único, formato válido |
| `senha` | texto | sim | mín. 6 caracteres, armazenar com hash |

## Endpoints

| Método | Rota | Autenticação | Descrição | Sucesso | Erro |
|---|---|---|---|---|---|
| POST | `/auth/registro` | não | Cadastrar usuário | 201 | 400, 409 |
| POST | `/auth/login` | não | Autenticar e receber token | 200 | 400, 401 |
| GET | `/usuarios/me` | JWT | Retornar dados do usuário logado | 200 | 401 |

## Exemplos de requisição

**Registrar usuário**

```bash
curl -X POST http://localhost:3000/auth/registro \
  -H "Content-Type: application/json" \
  -d '{"nome": "Ana Lima", "email": "ana@exemplo.com", "senha": "senha123"}'
```

```json
// 201 Created
{ "id": 1, "nome": "Ana Lima", "email": "ana@exemplo.com" }
```

**Fazer login**

```bash
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "ana@exemplo.com", "senha": "senha123"}'
```

```json
// 200 OK
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "tipo": "Bearer",
  "expiraEm": 3600
}
```

**Acessar rota protegida sem token**

```bash
curl http://localhost:3000/usuarios/me
```

```json
// 401 Unauthorized
{ "erro": "Token ausente ou inválido." }
```

## Qualidade do código

Organize em camadas:

```
controller  →  recebe a requisição e responde HTTP
service     →  gera token, valida senha, aplica regras
repository  →  acessa o banco de dados
```

## Critérios de avaliação

| Critério | Peso |
|---|---|
| Cadastro, login e rota protegida funcionando | 40% |
| Senha armazenada com hash (nunca texto puro) | 25% |
| Status HTTP corretos (401, 409, 400) | 20% |
| README com instruções de uso | 15% |

## Como entregar

1. Suba sua solução em um repositório público
2. Inclua um README no repositório explicando como rodar localmente
3. Abra uma [Discussion](https://github.com/Francisco-Montalvao/backend-challenges/discussions) com o link

> Nunca commite sua `JWT_SECRET` no repositório — use variável de ambiente e adicione `.env` ao `.gitignore`.

## Dicas finais

- Use bcrypt para hash — nunca MD5 ou SHA-1
- O token JWT deve expirar em 1 hora
- Email duplicado deve retornar `409 Conflict`, não `400`
- Refresh token e invalidação de logout são diferenciais, não requisitos

Boa sorte — este desafio aparece em quase toda entrevista backend!
