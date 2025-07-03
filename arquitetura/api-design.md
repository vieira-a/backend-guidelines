# Design de APIs REST

Este documento descreve o padrão que adoto para projetar APIs RESTful claras, consistentes, seguras e escaláveis. Ele reflete práticas consolidadas por experiência real em projetos backend com foco em qualidade técnica e facilidade de manutenção.

> Stack principal: Node.js, NestJS, PostgreSQL, Redis, Swagger, JWT, Jest

## Sumário

1. Nomeação de Recursos
2. Verbos HTTP
3. Versionamento
4. Formato de Requisições
5. Formato de Respostas
6. Tratamento de Erros
7. Paginação
8. Filtros e Ordenação
9. Autenticação e Autorização
10. Integração com APIs de Terceiros

---

## 1. Nomeação de Recursos

### O que é

Forma como os endpoints representam os recursos da aplicação.

### Como aplico

- Sempre nomeio o recurso no plural e com substantivos.
- Não utilizo verbos nas URLs.
- Tenho em mente que um recurso deve representar uma entidade do negócio.

- Correto: `/users`, `/orders`
- Incorreto: `/getUser`, `/createOrder`

### Benefícios

- Aderência ao padrão REST
- Clareza e previsibilidade para quem consome a API
- Facilita versionamento, documentação e testes

### Exemplo

GET /users/123

```json
{
  "data": {
    "id": "123",
    "name": "Darth Vader",
    "age": 75
  }
}
```

---

## 2. Verbos HTTP

### O que é

O método HTTP expressa a intenção da requisição em relação ao recurso.

### Como aplico

Utilizo os verbos HTTP conforme sua semântica:

| Verbo  | Uso                  |
| ------ | -------------------- |
| GET    | Leitura de recurso   |
| POST   | Criação de recurso   |
| PUT    | Atualização completa |
| PATCH  | Atualização parcial  |
| DELETE | Remoção de recurso   |

### Benefícios

- Compatibilidade com ferramentas REST
- Suporte a cache e idempotência
- Padronização no consumo da API

### Exemplo

PATCH /users/123

```json
{
  "email": "darth@sith.com"
}
```

---

## 3. Versionamento

### O que é

Controle da evolução da API por meio de versões distintas e compatíveis com múltiplos clientes.

### Como aplico

- Utilizo versionamento explícito via URL, no formato /v{n}.
- Mudanças incompatíveis com versões anteriores só são introduzidas em novas versões.

Exemplo: `/v1/users`, `/v2/users`

### Benefícios

- Transparência sobre alterações de contrato
- Permite múltiplas versões ativas
- Compatível com Swagger/OpenAPI

### Exemplo

GET /v1/products/42

```
{
  "data": {
    "id": 42,
    "name": "Lightsaber",
    "price": 249.90
  }
}
```

---

## 4. Formato de Requisições

### O que é

Padrão adotado para envio de dados nas requisições POST, PUT e PATCH.

### Como aplico

- Envio os dados no corpo da requisição em formato JSON.
- As chaves seguem a convenção camelCase.

### Benefícios

- Alinhamento com linguagens como JavaScript/TypeScript
- Facilidade de parsing no backend e frontend
- Compatível com validações automáticas

### Exemplo

```json
{
  "firstName": "Anderson",
  "email": "anderson@example.com"
}
```

---

## 5. Formato de Respostas

### O que é

Padrão de estrutura das respostas retornadas pela API.

### Como aplico

- Todas as respostas são envelopadas em um objeto com as chaves:
  - data: conteúdo principal da resposta
  - status: código HTTP (opcional)
  - message: descrição da operação (opcional)

### Benefícios

- Consistência entre endpoints
- Facilita tratamento no frontend
- Permite inclusão de metadados

Exemplo

```json
{
  "data": {
    "id": "abc123",
    "name": "Produto A"
  },
  "message": "Recurso retornado com sucesso",
  "status": 200
}
```

---

## 6. Tratamento de Erros

### O que é

Padrão para respostas de erro retornadas pela API.

### Como aplico

- Todas as respostas de erro seguem uma estrutura:

```json
{
  "error": {
    "message": "Campo 'email' é obrigatório",
    "code": "VALIDATION_ERROR"
  }
}
```

Utilizo os seguintes status HTTP:

| Código | Significado              |
| ------ | ------------------------ |
| 400    | Erro de validação        |
| 401    | Não autenticado          |
| 403    | Sem permissão            |
| 404    | Não encontrado           |
| 422    | Erro semântico           |
| 500    | Erro interno do servidor |

### Benefícios

- Facilita o diagnóstico e tratamento de falhas
- Padronização para frontend e integradores
- Permite rastreabilidade por tipo de erro

---

## 7. Paginação

### O que é

Estratégia para limitar grandes volumes de dados em respostas de listagem.

### Como aplico

- Utilizo os parâmetros `page` e `limit` como query string. A resposta inclui metadados no objeto meta.

### Benefícios

- Reduz o consumo de rede e memória
- Compatível com interfaces de paginação clássicas
- Fácil de implementar com SQL

### Exemplo

Requisição:

`GET /products?page=1&limit=10`

Resposta:

```json
{
  "data": [...],
    "meta": {
      "page": 1,
      "limit": 10,
      "total": 240
    }
}
```

## 8. Filtros e ordenação

### O que é

Permite retornar subconjuntos específicos de dados e controlar a ordem da listagem.

### Como aplico

- Utilizo query strings para aplicar filtros e ordenações:

`/users?role=admin&active=true`

`/products?category=eletronicos&sort=price_desc`

### Benefícios

- Flexibilidade para buscas dinâmicas
- Simples de usar no frontend
- Compatível com documentação Swagger

### Exemplo

`GET /orders?status=pending&sort=createdAt_desc`

---

## 9. Autenticação e Autorização

### O que é

Controle de acesso a recursos protegidos da API, garantindo que apenas usuários autenticados e autorizados possam acessar determinados endpoints.

### Como aplico

A estratégia de autenticação varia conforme o **tipo de cliente**.

#### Aplicações Web (navegador)

Utilizo **autenticação baseada em cookies** com os seguintes atributos:

- `HttpOnly`: evita leitura via JavaScript
- `Secure`: exige HTTPS
- `SameSite=Strict` ou `Lax`: mitiga CSRF

O token de sessão (ex: JWT ou session ID) é armazenado no cookie e enviado automaticamente pelo navegador.

### Benefícios

- Cookies com flags de segurança são mais resistentes a XSS
- Reduz exposição do token no frontend
- Flexibilidade para atender múltiplos tipos de cliente
- Pronto para ambientes com CORS configurado corretamente

**Considerações**

- Sempre usar HTTPS para que cookies com Secure funcionem.
- Sessões podem ter expiração com refresh token armazenado em cookie secundário ou mantido no servidor.
- Nunca armazeno tokens em localStorage ou sessionStorage em apps web.
- Sempre defino e documento claramente endpoints públicos .

---

## 10. Integração com APIs de Terceiros

### O que é

Interação entre a API principal da aplicação e serviços externos — como gateways de pagamento, plataformas de comunicação, serviços de armazenamento, entre outros.

### Como aplico

- Sigo princípios de segurança, rastreabilidade e isolamento.
- Faço chamadas externas de forma desacoplada, controlada e observável.

#### Estratégias adotadas:

- **Tokens secretos e chaves de API (API Keys)** armazenados em variáveis de ambiente seguras
- **Retries automáticos com backoff exponencial** para lidar com falhas transitórias
- **Timeouts curtos e controle de circuit breaker** para evitar travar a aplicação principal
- **Desacoplamento via filas** (ex: RabbitMQ, Redis, SQS) quando possível
- **Assinatura de webhooks** para garantir autenticidade das chamadas recebidas
- **Isolamento de responsabilidades** em módulos de integração específicos (ex: `PaymentsModule`, `EmailModule`, etc.)

### Autenticação com terceiros

- Uso **API Key** ou **Bearer Token** nos headers, conforme exigido pelo provedor.
- Tokens são obtidos via `client_credentials` ou `refresh_token` quando aplicável.
- Credenciais são armazenadas em `.env` e nunca comitadas no repositório.

Exemplo de header:

```http
Authorization: Bearer <external_token>
Content-Type: application/json
```

### Benefícios

- Evita falhas em cascata causadas por APIs externas indisponíveis
- Reduz risco de vazamento de credenciais
- Aumenta a rastreabilidade das integrações
- Garante que webhooks sejam legítimos e não forjados

**Considerações**

- Chamadas externas devem ser logadas e monitoradas
- Utilizo wrappers ou clients próprios por serviço sempre que possível (ex: StripeClient, WhatsAppService)
- Sempre valido dados de entrada mesmo que venham de fontes confiáveis
- Aplico testes bem definidos para integrações críticas

---

## Histórico de Alterações

| Data       | Versão | Autor           | Descrição                                      |
| ---------- | ------ | --------------- | ---------------------------------------------- |
| 2025-07-03 | 1.0.0  | Anderson Vieira | Criação inicial do guia de design de APIs REST |

---

## Autor

**Anderson Vieira**  
Desenvolvedor backend com foco em qualidade de software, arquitetura escalável e boas práticas em APIs REST.  
[GitHub](https://github.com/vieira-a) • [LinkedIn](https://linkedin.com/in/vieira-a)

---
