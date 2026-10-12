# Integração iOS com a PH API (Railway)

**Base URL atual:** `https://ph-api-production.up.railway.app`  
**Cliente iOS autorizado documentado:** Bundle ID `com.apple.mobile.MobileHouseArrest`, versão `2026.10.07`.

A documentação antiga do pacote `PH-api-project-2026.10.07.zip` aponta para `api-production-182c.up.railway.app`. O dashboard Railway atualmente identifica `ph-api-production.up.railway.app` como domínio do serviço `PH-API`. A rota de validação nesse domínio respondeu com JSON HTTP 400 para um corpo vazio, confirmando que a rota de API está ativa e que a requisição foi rejeitada por falta dos campos obrigatórios.

## Contrato móvel

As rotas abaixo recebem `POST` com `Content-Type: application/json` e o mesmo corpo:

```json
{
  "key": "CHAVE_DE_LICENCA",
  "device_id": "IDENTIFICADOR_ESTAVEL_DO_DISPOSITIVO",
  "package": "com.apple.mobile.MobileHouseArrest",
  "app_version": "2026.10.07"
}
```

| Método e rota | Finalidade |
|---|---|
| `POST /api/license/validate` | Validar a licença antes de liberar a interface protegida. |
| `POST /api/patches/catalog` | Obter o catálogo padrão de patches autorizado para a licença. |
| `POST /api/patches/external/catalog` | Obter o catálogo da seção `external`/Online. |
| `POST /api/announcement` | Obter o anúncio associado à licença. |
| `GET /api/announcement` | Consultar o anúncio público, sem contexto de licença. |
| `GET /api/patches/download?t=TOKEN` | Baixar um patch usando o `downloadUrl` temporário retornado pelo catálogo. |

Exemplo mínimo de validação:

```bash
curl -X POST "https://ph-api-production.up.railway.app/api/license/validate" \
  -H "Content-Type: application/json" \
  -d '{
    "key": "CHAVE_DE_LICENCA",
    "device_id": "IDENTIFICADOR_ESTAVEL_DO_DISPOSITIVO",
    "package": "com.apple.mobile.MobileHouseArrest",
    "app_version": "2026.10.07"
  }'
```

## Respostas e comportamento

- `200`: resposta válida; a validação deve exigir o booleano JSON `valid: true`.
- `400 invalid_request`: um ou mais campos obrigatórios estão ausentes ou vazios.
- `403 unsupported_client`: Bundle ID ou versão não correspondem ao build autorizado. Outras respostas `403` podem indicar licença inválida, expirada, revogada, limite de dispositivos ou download não autorizado.
- `503 service_unavailable`: indisponibilidade temporária do serviço, banco ou armazenamento.

Os catálogos retornam uma lista `patches`; os itens podem conter `slug`, `title`, `game`, `section`, `interfaceTab`, `fileName`, `sha256`, `sizeBytes` e `downloadUrl`. Usar o `downloadUrl` exatamente como retornado; ele contém um token temporário vinculado à licença, dispositivo, pacote e versão.

## Segurança e compatibilidade

- Não incluir senha administrativa, cookie de sessão, segredo do banco ou token privado do Railway no aplicativo ou neste documento. O app envia apenas a chave de licença fornecida ao usuário.
- A documentação antiga menciona `X-PH-IOS-Token`, mas o backend contido no pacote consultado implementa `hasValidIosToken()` como `true`; portanto esse header não deve ser tratado como autenticação efetiva.
- O backend valida a combinação de Bundle ID e versão permitida. Manter estes valores sincronizados com a configuração atual do serviço antes de publicar uma nova versão.
- Se o domínio for alterado no Railway, atualizar este documento e as URLs criptografadas do workflow `.github/workflows/ios-kyojinx.yml` em conjunto.
