# API atual — integração de validação de licença

**Atualizado em:** 12 de outubro de 2026  
**Escopo:** contrato de validação de licença. Este documento não descreve entrega ou download de patches.

## Estado deste projeto

O repositório contém uma demo SwiftUI de interface segura. Ela **não chama a API** e não contém um cliente de licença. O Bundle ID atual é `com.kaykemods733.phipademosafe` (`project.yml`). O servidor compara exatamente o `package` e o `app_version` recebidos com a combinação autorizada no backend. Portanto, não presuma que esta demo será autorizada: o Bundle ID dela não corresponde ao cliente iOS configurado no servidor.

Esta documentação não altera a lista de clientes permitidos no backend nem adiciona código de rede à demo. Uma futura integração requer que o responsável pelo backend autorize explicitamente o identificador e a versão apropriados.

## Endpoint

Base URL pública:

```text
https://api-production-182c.up.railway.app
```

Validação:

```text
POST /api/license/validate
```

Cabeçalhos:

```http
Content-Type: application/json
Accept: application/json
```

Corpo JSON — os quatro campos são obrigatórios e devem ser strings não vazias:

```json
{
  "key": "CHAVE_DE_LICENCA",
  "device_id": "IDENTIFICADOR_ESTAVEL_DO_DISPOSITIVO",
  "package": "BUNDLE_ID_DO_APP",
  "app_version": "VERSAO_DO_APP"
}
```

Exemplo de teste (use uma chave de teste válida criada no painel; não registre chaves reais em logs):

```bash
curl -i -X POST \
  'https://api-production-182c.up.railway.app/api/license/validate' \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json' \
  --data '{
    "key": "SUA_CHAVE_DE_TESTE",
    "device_id": "device-test-001",
    "package": "BUNDLE_ID_AUTORIZADO",
    "app_version": "VERSAO_AUTORIZADA"
  }'
```

## Respostas e decisão do cliente

Resposta autorizada contém `valid` como booleano `true`, por exemplo:

```json
{
  "valid": true,
  "code": "valid",
  "message": "Chave válida",
  "expiresAt": null,
  "activatedAt": null,
  "durationDays": 1,
  "durationMinutes": 1440,
  "deviceLimit": 1
}
```

Os campos de data e duração podem ser nulos ou omitidos conforme a licença. Trate-os como opcionais.

**Condição única para liberar a tela protegida:** resposta HTTP 200, JSON decodificado corretamente e `valid == true` (booleano). Texto de mensagem, campos ausentes, string `"true"`, resposta inesperada, falha de rede ou qualquer outro status não autorizam acesso. Em erros, mantenha o acesso fechado e mostre uma mensagem recuperável; não provoque crash.

Respostas/códigos relevantes:

| HTTP / `code` | Tratamento |
|---|---|
| `200`, `valid` | Prosseguir somente se `valid` for booleano `true`. |
| `200`, `invalid_key`, `revoked`, `expired`, `device_limit` | Negar acesso; não presumir que a licença está válida. |
| `400`, `invalid_request` | Corrigir os campos obrigatórios enviados. |
| `403`, `unsupported_client` | Package e versão não correspondem ao cliente autorizado pelo backend; negar acesso. |
| `503`, `service_unavailable` | Indisponibilidade temporária; não liberar acesso localmente. |
| Outros status, JSON inválido ou timeout | Tratar como falha fechada; não liberar acesso. |

## Segurança e privacidade

- Não coloque senha de administrador, cookies, credenciais do banco, segredos de sessão ou tokens de servidor dentro do app ou do repositório.
- A chave de licença é fornecida pelo usuário e enviada no corpo JSON. Não grave a chave completa em logs.
- Envie um identificador estável e específico do app, respeitando as regras de privacidade aplicáveis.
- O backend é a autoridade para licença e identidade do cliente; não confie apenas em estado local.
- O código atual mantém `X-PH-IOS-Token` por compatibilidade, mas a função correspondente no servidor não compara seu valor. **Não trate esse cabeçalho como autenticação nem o substitua pela validação de licença.**

## Compatibilidade com a demo

A demo deste repositório usa `com.kaykemods733.phipademosafe` e não implementa a chamada de validação. O servidor exige uma combinação autorizada de Bundle ID e versão; a combinação documentada para outro cliente não deve ser copiada para esta demo. Até o backend autorizar a identidade correta e uma integração ser implementada, esta página é referência documental — não significa que a demo esteja autenticada na API.

O workflow compila uma IPA **sem assinatura**. O arquivo Markdown acompanha o projeto-fonte e não é incorporado ao binário `.ipa`.
