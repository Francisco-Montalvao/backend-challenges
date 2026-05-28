# Por onde começar?

Autenticação tem várias partes móveis. Este arquivo explica cada uma na ordem certa para você não se perder.

## 1. Entendendo JWT

JWT não é criptografia — qualquer pessoa pode ler o payload. O que ele faz é **assinar** os dados para que você possa verificar que não foram alterados.

Estrutura de um token: `header.payload.signature`

Fluxo:
1. Usuário envia email + senha no login
2. Servidor verifica a senha e gera um token JWT assinado
3. Usuário envia o token no header `Authorization: Bearer <token>` em cada requisição protegida
4. Servidor valida a assinatura e identifica o usuário

Libs: `jsonwebtoken` (Node.js), `PyJWT` (Python), `jjwt` (Java), `golang-jwt/jwt` (Go)

## 2. Hash de senhas

Nunca salve a senha em texto puro. Use bcrypt — ele é lento de propósito, o que dificulta ataques de força bruta.

```
// Ao cadastrar
hash = bcrypt.hash(senhaDoUsuario, custoDeHash)
salvar hash no banco  ← nunca a senha original

// Ao fazer login
bcrypt.compare(senhaDigitada, hashSalvo)  →  true ou false
```

Libs: `bcrypt` (Node.js), `passlib[bcrypt]` (Python), Spring Security (Java), `golang.org/x/crypto/bcrypt` (Go)

## 3. Middleware de autenticação

A rota `/usuarios/me` precisa verificar o token antes de executar. O jeito mais organizado é criar um middleware:

```
requisição → [middleware verifica token] → controller
                        ↓ token inválido
                     retorna 401
```

O middleware lê o header `Authorization`, valida a assinatura do token e extrai o ID do usuário para passar ao controller.

## 4. Status HTTP para autenticação

| Situação | Status |
|---|---|
| Cadastro bem-sucedido | 201 |
| Login bem-sucedido | 200 |
| Token ausente ou inválido | 401 |
| Credenciais incorretas (senha errada) | 401 |
| Email já cadastrado | 409 |
| Dados inválidos (email malformado, campo faltando) | 400 |

## 5. Variáveis de ambiente

Sua `JWT_SECRET` nunca deve ir para o repositório. Use um arquivo `.env` e certifique-se de que ele está no `.gitignore` antes do primeiro commit:

```
JWT_SECRET=uma-string-longa-e-aleatoria-aqui
```

---

Links úteis:
- [JWT.io — Como funciona (em inglês)](https://jwt.io/introduction)
- [MDN — HTTP Status Codes (pt-BR)](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Status)
