# Z-API + Supabase — WhatsApp Message Sender

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![Supabase](https://img.shields.io/badge/Supabase-2.10.0-green)
![Z-API](https://img.shields.io/badge/WhatsApp-Z--API-brightgreen)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

Script Python para disparos de WhatsApp em lote — busca contatos ativos no Supabase, aplica o template escolhido, registra o resultado no banco e respeita um intervalo entre envios para evitar bloqueios.

---

## Funcionalidades

- **Templates de mensagem** configuráveis via CLI, sem editar código
- **Registro de envios** no banco: `sent_at`, `last_status`, `last_template`, `last_attempted_at`
- **Filtro de pendentes** — só processa contatos que ainda não receberam mensagem (`sent_at IS NULL`)
- **Dry-run** para simular disparos sem chamar a Z-API nem gravar no banco
- **Delay configurável** entre envios para não acionar filtros de spam
- **Validação de telefone** antes de qualquer chamada à API

---

## Pré-requisitos

- Python 3.9 ou superior
- Conta ativa na [Z-API](https://www.z-api.io/) com instância WhatsApp conectada
- Projeto criado no [Supabase](https://supabase.com/)

---

## Configuração do banco

Execute o arquivo `schema.sql` no SQL Editor do Supabase:

```sql
CREATE TABLE contacts (
    id                 BIGSERIAL PRIMARY KEY,
    name               TEXT        NOT NULL,
    phone              TEXT        NOT NULL,
    active             BOOLEAN     NOT NULL DEFAULT TRUE,
    sent_at            TIMESTAMPTZ,
    last_attempted_at  TIMESTAMPTZ,
    last_template      TEXT,
    last_status        TEXT
);

CREATE INDEX idx_contacts_pending ON contacts (active, sent_at)
    WHERE active = TRUE AND sent_at IS NULL;
```

O campo `phone` aceita apenas dígitos, sem `+`, espaços ou traços (DDI + DDD + número).
Formato esperado: `5511912345678`

---

## Instalação

```bash
git clone https://github.com/eduardotorres672/zapi-supabase-whatsapp.git
cd zapi-supabase-whatsapp

python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate

pip install -r requirements.txt

cp env.example .env
```

Edite o `.env` com suas credenciais:

```env
# Supabase
SUPABASE_URL=https://seusupabase.supabase.co
SUPABASE_KEY=sua_anon_key

# Z-API
ZAPI_INSTANCE_ID=seu_instance_id
ZAPI_TOKEN=seu_token
ZAPI_CLIENT_TOKEN=seu_client_token
```

As credenciais Z-API estão disponíveis no painel da instância em [app.z-api.io](https://app.z-api.io).

---

## Uso

**Disparo padrão:**

```bash
python main.py
```

**Com template e limite específicos:**

```bash
python main.py --template promocao --limit 100 --delay 3
```

**Dry-run — simula sem enviar nem gravar:**

```bash
python main.py --template lembrete --dry-run
```

**Todos os parâmetros disponíveis:**

| Parâmetro | Padrão | Descrição |
|---|---|---|
| `--template` | `default` | Template de mensagem: `default`, `promocao`, `lembrete`, `reativacao` |
| `--limit` | `50` | Máximo de contatos por execução |
| `--delay` | `2.0` | Intervalo em segundos entre cada envio |
| `--dry-run` | — | Simula sem chamar Z-API nem gravar no banco |

---

## Templates disponíveis

| Template | Mensagem |
|---|---|
| `default` | Olá, {name}! Tudo bem com você? |
| `promocao` | Oi, {name}! Temos uma novidade especial esperando por você. |
| `lembrete` | Olá, {name}. Passando para lembrar do seu compromisso. |
| `reativacao` | Sentimos sua falta, {name}! Que tal a gente retomar o contato? |

Para adicionar novos templates, edite o dicionário `TEMPLATES` no início de `main.py`.

---

## Saída esperada

```
2025-01-15 10:32:01 [INFO] 3 contato(s) encontrado(s) | template: promocao | delay: 2.0s
2025-01-15 10:32:03 [INFO] enviado → Eduardo (5511999999991)
2025-01-15 10:32:05 [INFO] enviado → Victoria (5511999999992)
2025-01-15 10:32:07 [INFO] enviado → Carlos (5511999999993)
2025-01-15 10:32:07 [INFO] concluído — 3 enviado(s), 0 falha(s)
```

---

## Agendamento

Para rodar automaticamente (ex: todo dia às 9h), adicione ao cron:

```bash
crontab -e
```

```
0 9 * * * /caminho/para/.venv/bin/python /caminho/para/main.py --template default
```

Ou via **GitHub Actions** criando um workflow com `schedule: cron`.

---

## Stack

- [Python 3](https://www.python.org/)
- [supabase-py](https://pypi.org/project/supabase/) `2.10.0`
- [Z-API](https://www.z-api.io/)
- [python-dotenv](https://pypi.org/project/python-dotenv/) `1.0.1`
- [requests](https://pypi.org/project/requests/) `2.32.3`

---

## Licença

[MIT](LICENSE)
