# Monitor de Pós-Graduação Gratuita — Wanderson

Painel web que reúne **mestrados, especializações, MBAs e cursos longos gratuitos** (públicos ou com bolsa integral) nas áreas de economia, finanças, contabilidade, auditoria, administração, dados, IA e gestão pública — **online (EAD)** ou **presencial em São Paulo / Grande SP**.

Espelha a arquitetura do monitor da Jully: dados separados em `events.json`, painel estático em `index.html`, atualização automática por GitHub Actions e publicação no GitHub Pages.

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | Painel (carrega `events.json`); filtros, **★ favoritar**, **✕ não me interessa**, prioridades |
| `sobre.html` | Página "Como funciona" |
| `events.json` | Base de dados dos programas (a fonte da verdade) |
| `last_update.json` | Metadados da última rodada (data, contadores) |
| `update.py` | Robô que recalcula prazos, agrega novas fontes e envia e-mail |
| `sources.json` | Lista de fontes (feeds RSS + buscas) — editável |
| `.github/workflows/update.yml` | Agenda a execução (segunda 9h BRT) |
| `requirements.txt` | Dependência Python (`feedparser`) |

## Rodar localmente

O painel precisa ser servido por HTTP (o `fetch` do `events.json` não funciona abrindo o arquivo direto do disco):

```bash
cd monitor-pos-web
python -m http.server 8000
# abra http://localhost:8000
```

Para rodar o agregador uma vez:

```bash
pip install -r requirements.txt
python update.py
```

Sem nenhuma variável de ambiente, ele só recalcula prazos (abre/encerra). As camadas extras são opcionais (ver abaixo).

## Publicar no GitHub Pages

1. Crie um repositório (ex.: `monitor-pos-web`) e suba estes arquivos.
2. **Settings → Pages** → Source: *Deploy from a branch* → branch `main`, pasta `/ (root)` → Save.
3. O painel fica em `https://SEU-USUARIO.github.io/monitor-pos-web/`.
4. **Settings → Actions → General** → marque *Read and write permissions* (para o robô commitar o `events.json`).

A partir daí, o GitHub Actions roda toda segunda às 9h (e pelo botão **Run workflow**), atualiza os dados e republica sozinho.

## Variáveis (Secrets) opcionais

Em **Settings → Secrets and variables → Actions**:

| Secret | Para quê | Obrigatório? |
|---|---|---|
| `ANTHROPIC_API_KEY` | Curadoria por IA (filtra/enriquece os candidatos novos) | Não |
| `GOOGLE_API_KEY` + `GOOGLE_CSE_ID` | Busca web ampla (Google Custom Search) | Não |
| `GMAIL_USER` + `GMAIL_APP_PASSWORD` + `MAIL_TO` | Envio do e-mail digest | Não |

Sem nenhum deles, o monitor continua funcionando com a base curada e os feeds RSS — apenas sem busca ampla, sem curadoria por IA e sem e-mail.

## Como adicionar/editar um programa à mão

Edite `events.json`. Cada item segue o formato:

```json
{
  "id": "slug-unico",
  "nome": "Nome do programa",
  "inst": "Instituição",
  "tipo": "Mestrado | Especialização | MBA | Curso longo",
  "areas": ["economia", "finanças"],
  "mod": "online | sp | hibrido | presencial-fora-sp | varia",
  "modLabel": "Texto da modalidade",
  "local": "Cidade/UF (se presencial)",
  "prazo": "Texto do prazo",
  "prazoSort": "2026-07-31",
  "status": "open | soon | monitor | always | closed",
  "obs": "Observações",
  "link": "https://edital-oficial"
}
```

O `status` de itens com `prazoSort` é recalculado a cada rodada: vira `closed` quando a data passa. Itens `always`, `monitor` e `soon` não são alterados automaticamente.

## Regra de ouro

Só entra o que é **gratuito** (público ou bolsa integral) e do perfil. Programas com mensalidade são descartados, exceto com bolsa integral confirmada. Cursos Coursera/edX entram como **Financial Aid**, não como gratuito puro.
