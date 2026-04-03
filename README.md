# 🏛️ Monitor Matérias Legislativas — Câmara Municipal de Rio Branco/AC

Monitora automaticamente a API do SAPL da Câmara Municipal de Rio Branco e envia email quando há matérias legislativas novas. Roda **4x por dia** via GitHub Actions (8h, 12h, 17h e 21h, horário de Brasília).

---

## Como funciona

1. O GitHub Actions roda o script nos horários configurados
2. O script chama a API pública do SAPL (`sapl.riobranco.ac.leg.br/api`)
3. Compara as matérias recebidas com as já registradas no `estado.json`
4. Se há matérias novas → envia email com a lista organizada por tipo
5. Salva o estado atualizado no repositório

---

## Sistema utilizado

**SAPL Interlegis** — sistema open source do Senado Federal, versão 3.1.165-RC1.

```
URL base: https://sapl.riobranco.ac.leg.br
Endpoint: GET /api/materia/materialegislativa/
Params:   ?ano=2026&page=1&page_size=100&ordering=-id
Docs:     https://sapl.riobranco.ac.leg.br/api/schema/swagger-ui/
```

API pública, sem autenticação.

---

## Estrutura do repositório

```
monitor-proposicoes-rb/
├── monitor.js                      # Script principal
├── package.json                    # Dependências (só nodemailer)
├── estado.json                     # Estado salvo automaticamente pelo workflow
├── README.md                       # Este arquivo
└── .github/
    └── workflows/
        └── monitor.yml             # Workflow do GitHub Actions
```

---

## Setup — Passo a Passo

### PARTE 1 — Preparar o Gmail

**1.1** Acesse [myaccount.google.com/security](https://myaccount.google.com/security)

**1.2** Certifique-se de que a **Verificação em duas etapas** está ativa.

**1.3** Busque por **"Senhas de app"** e clique.

**1.4** Digite o nome `monitor-riobranco` e clique em **Criar**.

**1.5** Copie a senha de **16 letras** — ela só aparece uma vez.

> Se já tem App Password de outro monitor, pode reutilizar.

---

### PARTE 2 — Criar o repositório no GitHub

**2.1** Acesse [github.com](https://github.com) → **+ → New repository**

**2.2** Preencha:
- **Repository name:** `monitor-proposicoes-rb`
- **Visibility:** Private

**2.3** Clique em **Create repository**

---

### PARTE 3 — Fazer upload dos arquivos

**3.1** Clique em **"uploading an existing file"**

**3.2** Faça upload de:
```
monitor.js
package.json
README.md
```
Clique em **Commit changes**.

**3.3** O `monitor.yml` precisa de pasta específica. Clique em **Add file → Create new file**, digite:
```
.github/workflows/monitor.yml
```
Cole o conteúdo do arquivo `monitor.yml`. Clique em **Commit changes**.

---

### PARTE 4 — Configurar os Secrets

**4.1** No repositório: **Settings → Secrets and variables → Actions**

**4.2** Clique em **New repository secret** e crie os 3 secrets:

| Name | Valor |
|------|-------|
| `EMAIL_REMETENTE` | seu Gmail (ex: seuemail@gmail.com) |
| `EMAIL_SENHA` | a senha de 16 letras do App Password (sem espaços) |
| `EMAIL_DESTINO` | email onde quer receber os alertas |

---

### PARTE 5 — Testar

**5.1** Vá em **Actions → Monitor Matérias Rio Branco/AC → Run workflow → Run workflow**

**5.2** Aguarde ~15 segundos. Verde = funcionou.

**5.3** O **primeiro run** envia email com as 100 matérias mais recentes do ano e salva o estado. A partir do segundo run, só envia se houver novidades.

---

## Email recebido

```
🏛️ Câmara de Rio Branco/AC — 3 nova(s) matéria(s) legislativa(s)

Indicação — 2 matéria(s)
  150/2026 | -  | 02/04/2026 | Indica pavimentação...
  149/2026 | -  | 02/04/2026 | Indica iluminação...

Projeto de Lei Ordinária — 1 matéria(s)
  42/2026  | -  | 02/04/2026 | Dispõe sobre...
```

> **Nota sobre autores:** o SAPL não retorna o autor inline na listagem da API — seria necessária
> uma chamada extra por matéria (inviável em escala). Para ver o autor, clique na matéria no portal.

---

## Horários de execução

| Horário BRT | Cron UTC |
|-------------|----------|
| 08:00       | 0 11 * * * |
| 12:00       | 0 15 * * * |
| 17:00       | 0 20 * * * |
| 21:00       | 0 0 * * *  |

---

## Resetar o estado

1. No repositório, clique em `estado.json` → lápis
2. Substitua o conteúdo por:
```json
{"proposicoes_vistas":[],"ultima_execucao":""}
```
3. Commit → rode o workflow manualmente

---

## Problemas comuns

**Não aparece "Senhas de app" no Google**
→ Ative a verificação em duas etapas primeiro.

**Erro "Authentication failed" no log**
→ Verifique se `EMAIL_SENHA` foi colado sem espaços.

**Workflow não aparece em Actions**
→ Confirme que o arquivo está em `.github/workflows/monitor.yml`.

**Log mostra "0 matérias encontradas"**
→ A API pode estar fora do ar. Teste no browser:
`https://sapl.riobranco.ac.leg.br/api/materia/materialegislativa/?ano=2026&page=1&page_size=1`
