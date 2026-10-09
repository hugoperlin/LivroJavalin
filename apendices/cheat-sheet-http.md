# Apêndice A: Cheat Sheet do Protocolo HTTP

Um guia rápido de consulta para o estudante de desenvolvimento backend e APIs REST.

---

## 1. Verbos HTTP (Métodos de Requisição)

| Verbo | Tem Corpo (Body)? | Idempotente? | Seguro (Safe)? | Finalidade Típica |
| :--- | :---: | :---: | :---: | :--- |
| **GET** | Não | Sim | Sim | Recuperar dados sem alterar o estado do servidor. |
| **POST** | Sim | Não | Não | Criar um novo recurso subordinado ou processar dados. |
| **PUT** | Sim | Sim | Não | Substituir/atualizar completamente um recurso existente. |
| **PATCH** | Sim | Não | Não | Atualizar parcialmente campos específicos de um recurso. |
| **DELETE**| Opcional | Sim | Não | Remover permanentemente um recurso do sistema. |

> **O que significa "Idempotente"?**
> Uma operação é idempotente quando executá-la uma única vez produz exatamente o mesmo efeito no sistema que executá-la 100 vezes consecutivas (ex: `DELETE /usuarios/5`). O `POST` não é idempotente porque executá-lo 10 vezes criará 10 registros distintos.

---

## 2. Códigos de Status HTTP (*Status Codes*)

Os códigos de status são números inteiros de 3 dígitos divididos em 5 famílias pela primeira casa decimal:

### Família 2xx: Sucesso (*Success*)
* **`200 OK`**: A requisição foi bem-sucedida. Padrão para respostas de `GET` e `PUT`.
* **`201 Created`**: O recurso foi criado com sucesso. Padrão para respostas de `POST`.
* **`204 No Content`**: A ação foi realizada com sucesso, mas não há corpo de resposta a ser retornado (comum em operações `DELETE`).

### Família 3xx: Redirecionamento (*Redirection*)
* **`301 Moved Permanently`**: O recurso mudou de endereço definitivamente.
* **`304 Not Modified`**: O conteúdo não mudou desde a última consulta em cache.

### Família 4xx: Erro do Cliente (*Client Error*)
* **`400 Bad Request`**: A requisição é inválida ou malformada (ex: JSON com sintaxe quebrada ou parâmetro obrigatório ausente).
* **`401 Unauthorized`**: Falta de autenticação (o usuário precisa se identificar).
* **`403 Forbidden`**: Autenticado, mas sem permissão de acesso ao recurso específico.
* **`404 Not Found`**: O recurso solicitado não existe no servidor.
* **`409 Conflict`**: Violação de regra de unicidade (ex: tentar cadastrar um e-mail que já existe).
* **`422 Unprocessable Entity`**: Sintaxe válida, mas erro semântico de validação nos campos.

### Família 5xx: Erro do Servidor (*Server Error*)
* **`500 Internal Server Error`**: Ocorreu uma exceção não tratada ou erro inesperado no código Java.
* **`502 Bad Gateway`**: O servidor intermediário recebeu uma resposta inválida.
* **`503 Service Unavailable`**: Servidor sobrecarregado ou temporariamente em manutenção.

---

## 3. Anatomia de uma Mensagem HTTP

### Requisição (*HTTP Request*)
```http
POST /usuarios HTTP/1.1
Host: localhost:8080
Content-Type: application/json
User-Agent: PostmanRuntime/7.36.0

{
  "nome": "Carla Mendes",
  "email": "carla@tads.edu.br",
  "perfil": "ALUNO"
}
```

### Resposta (*HTTP Response*)
```http
HTTP/1.1 201 Created
Date: Wed, 25 Sep 2026 12:00:00 GMT
Content-Type: application/json; charset=UTF-8

{
  "id": 1,
  "nome": "Carla Mendes",
  "email": "carla@tads.edu.br",
  "perfil": "ALUNO"
}
```

---

## 4. Guia Rápido dos Métodos de `io.javalin.http.Context`

| Ação Desejada | Método no Javalin |
| :--- | :--- |
| Capturar parâmetro da URL (`/recurso/{id}`) | `int id = Integer.parseInt(ctx.pathParam("id"));` |
| Capturar query da URL (`/busca?termo=java`) | `String busca = ctx.queryParam("termo");` |
| Ler o corpo como texto puro | `String corpo = ctx.body();` |
| Converter JSON para Objeto Java | `Usuario u = ctx.bodyAsClass(Usuario.class);` |
| Definir código de status HTTP | `ctx.status(201);` |
| Responder texto plano | `ctx.result("Operação concluída!");` |
| Responder objeto Java como JSON | `ctx.json(meuObjeto);` |
| Definir cabeçalho de resposta | `ctx.header("X-App-Version", "1.0");` |
