---
description: >-
  O Webhook Manager é um sistema responsável por gerenciar o envio e recebimento
  de webhooks entre sistemas distribuídos. Ele permite registrar endpoints,
  configurar eventos, monitorar entregas e reproc
---

# Webhook Manager — Documentação

## Conceitos básicos

### O que é um webhook?

Um **webhook** é uma requisição HTTP enviada automaticamente quando um evento ocorre em um sistema.

Por exemplo:

* um pagamento aprovado
* um usuário criado
* um pedido atualizado

Quando isso acontece, o sistema envia uma requisição para uma URL configurada.

Exemplo de requisição:

```
POST https://api.exemplo.com/webhook

{
  "event": "payment.approved",
  "data": {
    "paymentId": "12345",
    "amount": 150.00
  }
}
```

***

## Está tudo bem?

## Arquitetura do sistema

O Webhook Manager possui os seguintes componentes:

#### Event Publisher

Responsável por disparar eventos internos do sistema.

#### Webhook Registry

Banco de dados onde ficam registrados:

* URLs de webhook
* eventos assinados
* status da integração

#### Delivery Service

Serviço responsável por enviar os webhooks para os endpoints registrados.

#### Retry Queue

Fila responsável por reprocessar webhooks que falharam.

#### Delivery Logs

Registro histórico de todas as entregas de webhook.

***

## Criando um webhook

Para registrar um webhook siga os passos abaixo.

#### Passo 1 — Criar o endpoint

Acesse o painel administrativo e clique em:

```
Webhooks → Create Webhook
```

***

#### Passo 2 — Informar os dados

Preencha os campos:

| Campo      | Descrição                       |
| ---------- | ------------------------------- |
| Name       | Nome da integração              |
| URL        | Endpoint que receberá o webhook |
| Events     | Eventos que serão enviados      |
| Secret Key | Chave usada para assinatura     |

Exemplo:

```
Name: Payment Integration
URL: https://api.cliente.com/webhooks/payment
Events:
- payment.created
- payment.approved
```

***

#### Passo 3 — Salvar

Após salvar, o webhook estará ativo e pronto para receber eventos.

***

## Eventos disponíveis

O sistema suporta os seguintes eventos:

| Evento           | Descrição           |
| ---------------- | ------------------- |
| user.created     | Novo usuário criado |
| user.updated     | Usuário atualizado  |
| payment.created  | Pagamento iniciado  |
| payment.approved | Pagamento aprovado  |
| payment.failed   | Pagamento falhou    |
| order.created    | Pedido criado       |

***

## Estrutura do payload

Todos os webhooks seguem o mesmo formato.

```
{
  "id": "evt_8293012",
  "event": "payment.approved",
  "timestamp": "2026-03-05T15:00:00Z",
  "data": {
    "paymentId": "12345",
    "amount": 150.00,
    "currency": "BRL"
  }
}
```

***

## Segurança

Os webhooks podem ser assinados utilizando uma **Secret Key**.

O sistema envia um header chamado:

```
X-Webhook-Signature
```

Esse valor é um **HMAC SHA256** do payload.

Exemplo de header:

```
X-Webhook-Signature: 9c1a23bd7810f12a...
```

O endpoint receptor deve validar essa assinatura.

***

## Monitoramento de entregas

Todas as chamadas de webhook são registradas.

Para visualizar:

```
Webhooks → Delivery Logs
```

Cada registro contém:

* status HTTP
* tempo de resposta
* payload enviado
* tentativa de envio

***

## Política de retry

Quando um webhook falha, o sistema agenda novas tentativas.

A política padrão é:

| Tentativa | Intervalo   |
| --------- | ----------- |
| 1         | imediato    |
| 2         | 30 segundos |
| 3         | 5 minutos   |
| 4         | 30 minutos  |
| 5         | 2 horas     |

Após 5 falhas o webhook é marcado como **Failed**.

***

## Reprocessando um webhook

Para reenviar manualmente um webhook:

```
Webhooks → Delivery Logs → Retry
```

Isso enviará novamente o payload original.

***

## Boas práticas

Para garantir uma integração confiável:

* responda com **HTTP 200 rapidamente**
* processe o webhook de forma **assíncrona**
* implemente **idempotência**
* valide sempre a **assinatura**

***

## Perguntas frequentes

#### O que acontece se meu endpoint estiver offline?

O Webhook Manager tentará reenviar o webhook de acordo com a política de retry.

***

#### Posso registrar múltiplos endpoints?

Sim. Um mesmo evento pode ser enviado para múltiplos endpoints.

***

#### Existe limite de webhooks?

Depende do plano utilizado.



