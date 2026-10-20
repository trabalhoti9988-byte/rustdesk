# Aurum Nexus V2 (1.4.9-7) — pacote de atualização

**Para:** o Claude que for aplicar isto no Atrio (máquina do Dr. Rodrigo).
**De:** Rafael Ferreira (TI). Preparado com Claude em 19-20/10/2026.

## Conteúdo da pasta
- `AurumNexus-1.4.9-7-x86_64.exe` — instalador pronto da V2.
  - Compilado e testado no PC do Rafael.
  - Já traz embutidos o servidor `aurum.fagundesadv.com.br` e a chave dele: **não precisa** de `host=` no nome do arquivo.
- `0001…0009-*.patch` — os 9 commits da V2 em formato `git format-patch`, em cima de `8f91d53` (build-6), branch `aurum` de `trabalhoti9988-byte/rustdesk`.
- Este arquivo.

## O que mudou na V2

| # | Mudança | Por quê |
|---|---|---|
| 0001 | Servidor embutido = `aurum.fagundesadv.com.br`; padrões `direct-server=Y` e `image_quality=best`; o instalador recusa rodar de dentro da pasta instalada | O .90 foi desligado. Conexão direta na rede local. O modo "balanced" borrava o texto. Instalar de dentro da pasta apagou 86 arquivos em 29/07 |
| 0002 | Aviso de atualização pelo nosso servidor, com o cartão "Nova versão" e o botão Atualizar | Veja "Atualizações futuras" abaixo |
| 0003 | Opção local `aurum-versao-url` | Só para testar o aviso apontando para outro servidor |
| 0004 | Senha como no AnyDesk (`verification-method=use-permanent-password`) | Sem senha criada, só entra com Aceitar. Com a senha do dono, ID + senha entra direto |
| 0005 | "Aurum Nexus V2" em Aplicativos instalados e na tela inicial | O `APP_NAME` não muda: dele saem a pasta de configuração, o ID e o título da janela |
| 0006 | Esconder "Parar serviço" (`hide-stop-service`) | O clique **desinstala** o serviço e a máquina some do Aurum. Aconteceu de verdade em 19/10 |
| 0007 | Tirar o guard `is_custom_client()` do aviso | Ele é `app_name != "RustDesk"`, então é sempre verdadeiro no Aurum e desligava o aviso |
| 0008 | `is_msi_installed` pelo `APP_ID` | Procurava `Uninstall\Aurum Nexus`, a chave não existia e o botão Atualizar falhava |
| 0009 | O download só é aceito do nosso servidor | O original só aceitava `github.com/rustdesk/rustdesk` e recusava o nosso endereço em silêncio |

Versão: `1.4.9-7` (no `Cargo.toml`, no `Cargo.lock` e no `aurum.yml`). O `VersionBuild` do registro pega só o `9`.

## Testado no PC do Rafael (20/10)
- ✅ Registra no Atrio, com latência de 1-3 ms.
- ✅ Conexão direta pela rede local nos dois sentidos, sem relay (porta 21118 aberta).
- ✅ H265 pela placa Intel (QSV) quando ela existe. Numa máquina com GT 610, cai no AV1 por software.
- ✅ O Matheus entrou no PC do Rafael com ID + senha do dono. Sem senha, só com Aceitar.
- ✅ "Aurum Nexus V2" em Aplicativos instalados (versão 1.4.9.7) e na tela inicial.
- ✅ Instalar por cima da V1 mantém a configuração.
- ✅ Instalar de dentro da pasta instalada: **recusado, sem apagar nada** (102 arquivos antes e depois, sem reinstalar o serviço).
- ✅ Aviso de atualização com servidor falso: o cartão "Nova versão (1.4.9-8)" apareceu, a checagem de MSI passou e o endereço foi aceito.
- ⏳ **Download e reinstalação pelo botão:** só dá com **HTTPS**. O cliente de download recusa HTTP (`Strict HTTP client requires HTTPS`), e o servidor falso era HTTP. **É o primeiro teste a fazer no Atrio** (ver abaixo).

## Tarefa no Atrio — obrigatória para o aviso de atualização funcionar
Hoje `https://aurum.fagundesadv.com.br` responde **401 "sem sessao"** em tudo. O app **não tem login**, então estas rotas têm que ser **públicas**:

1. `GET /api/versao` → `{"url": "https://aurum.fagundesadv.com.br/atualizacao/tag/1.4.9-7"}`
   - Só o campo `url`. A versão é o que vem depois da última `/`.
   - O app só mostra o cartão quando essa versão é **maior** que a dele.
2. `GET` **e** `HEAD` `/atualizacao/download/<versao>/AurumNexus-<versao>-x86_64.exe` → o arquivo.
   - O `HEAD` tem que devolver `Content-Length`.
   - No FastAPI: `@app.api_route(..., methods=["GET", "HEAD"])` + `FileResponse`.
   - **Tem que ser HTTPS, na mesma origem `https://aurum.fagundesadv.com.br`**, sem porta explícita, query nem `#`. Qualquer outra coisa é recusada pelo app de propósito (`updater.rs`, `get_update_download_file_from_url`).

**Primeiro teste no Atrio:**
1. Publique `1.4.9-8` apontando para o mesmo exe, renomeado para `AurumNexus-1.4.9-8-x86_64.exe`.
2. Numa máquina com a V2, abra o Aurum e clique em Atualizar. Ele deve baixar, pedir o UAC e reinstalar.
3. Depois volte o `/api/versao` para `1.4.9-7`.

## Atualizações futuras (vale para TODAS as próximas versões)
Depois que as máquinas tiverem a V2, publicar uma versão nova é sempre:
1. **Subir a versão** no `Cargo.toml` (e no `Cargo.lock`, linha do pacote `rustdesk`, porque o build usa `--locked`) e no `VERSION` do `aurum.yml`: `1.4.9-8`, `1.4.9-9`, `1.4.10`…
   - **Se a versão não subir, ninguém recebe o aviso.**
   - Comparação: `1.4.9-7` < `1.4.9-8` < `1.4.10` (`hbb_common::get_version_number`).
2. **Compilar:** no GitHub, pela tag `build-<n>`, ou no PC do Rafael com `F:\aurum-toolchain\compilar.ps1` (~10 min).
3. **No Atrio:** ponha o exe em `/atualizacao/download/<versao>/` e troque a versão no `/api/versao`.
4. Cada máquina mostra o cartão quando a pessoa abre o Aurum. Nada se instala sozinho: `allow-auto-update` continua desligado.

A **V1 (build-6) não tem o aviso**, então as máquinas com a V1 precisam receber a V2 na mão **uma vez**.

## Conferir antes de distribuir
- A chave embutida é `WYW9MeOO8NUNksg2B+nm+zCAXcMNGgr1gvJjqL2vi7c=`. O Atrio respondeu "ID não existe" (e não "chave diferente") com ela, mas confira o `id_ed25519.pub` do hbbs.
- Confira se o hbbs roda com `-r <relay>`. Sem isso, o cliente pode esperar 18 s antes de cair no relay.
- **Mudança de uso para a TI:** depois da V2, o acesso sem ninguém na frente da máquina só funciona onde alguém **criou a senha permanente**.
- Na instalação, a máquina pode ganhar **ID novo** (o PC do Rafael passou a 320 889 966).

## Como aplicar os patches e compilar pelo GitHub
```bash
git checkout aurum                 # tem que estar em 8f91d53
git am /caminho/00*.patch
git -C libs/hbb_common apply --check ../../.github/aurum/hbb_common.patch
git push aurum refs/heads/aurum:refs/heads/aurum
git tag build-7 && git push aurum refs/tags/build-7
```
**Não publique release:** o repositório é público e o exe leva o endereço e a chave do servidor.

## Como reverter
- **Código:** `git revert` dos commits.
- **Máquina:** reinstale o instalador da build-6. O backup da pasta instalada do PC do Rafael está em `F:\aurum-build\backup-programfiles`.

## Fica para depois
- Furo de NAT por UDP/IPv6 (`enable-udp-punch`), que depende da versão do hbbs.
- Log de sessão e comentário no fim (`api-server` + rotas). Hoje o app tenta `:21114/api/heartbeat` a cada 30 s e falha: é só ruído no log.
- O `BUILD_DATE` da build local não é atualizado quando o `Cargo.toml` não muda. No CI ele sai certo.

Estudo de origem: `F:\ESTUDO-ANYDESK.md` e `F:\COMPARATIVO-AURUM-ANYDESK.md`.
