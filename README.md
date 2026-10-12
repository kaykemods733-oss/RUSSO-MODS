# RUSSO-MODS

## PH iOS UI Demo — projeto seguro

Este repositório contém somente uma demo SwiftUI de interface. Não inclui o fonte explorável nem o workflow original.

O pacote-fonte mais recente é **PH-IPA-UI-demo-safe-v2.zip**. Ele inclui o projeto XcodeGen e a documentação atual de validação da API em **API_INTEGRATION.md**. O arquivo PH-IPA-UI-demo-safe.zip é a versão anterior; use o v2.

A demo atual não chama a API. O guia documenta o contrato de validação, mas deixa claro que o Bundle ID desta demo não é o cliente autorizado pelo backend. Nenhuma lista permitida do servidor foi alterada.

O workflow .github/workflows/build-demo-ipa.yml é manual-only e produz uma IPA sem assinatura como artifact, retido por sete dias. Ele não publica Releases. A documentação acompanha o projeto-fonte; ela não é incorporada ao binário IPA.

Para compilar, abra a aba Actions, selecione **Build safe UI demo IPA** e use **Run workflow**.
