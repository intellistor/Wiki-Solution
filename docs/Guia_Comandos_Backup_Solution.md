# 📌 Intellistor Backup Solution – Comandos Essenciais (MVP)

Este guia consolida todos os comandos relevantes, na ordem correta, para uso do SDK e do Agent, cobrindo:
- criação de policy
- execução manual de backup
- restore (completo e seletivo)
- execução governada via Agent
- agendamento automático no Windows
- validações operacionais e de auditoria

---
## 1️⃣ SDK – Core Engine (backup-client)

> Usado para desenvolvimento, testes locais e operações técnicas diretas.
> Não gera logs persistidos nem envia logs ao S3.

---
### 1.1 Criar uma Policy
Define o que será protegido, como e quando.

```bash
backup-client policy-create --out policies/policy_01.yaml --paths "C:\workload\git_lab\docs\imagens" --schedule "0 2 * * *"
```

📌 Observações:
- paths → diretórios incluídos no backup
- schedule → cron informativo (usado depois pelo Agent)
- Defaults:
    - paths: diretório atual
    - schedule: 0 2 * * *

---
### 1.2 Executar Backup (SDK direto)
```bash
backup-client backup --policy policies\policy_01.yaml --out .\out
```

🔎 Resultado:
- Cria backup criptografado (backup.tar.gz.enc)
- Envia objeto de backup para o S3
- Envia manifest.json para o S3
- Retorna JSON com:
    - backup_id
    - timestamp
    - s3.key_backup
    - s3.key_manifest
    - cli_restore (comando pronto)

⚠️ Importante:
> Logs somente no console
> Sem rastreabilidade operacional

---
### 1.3 Restore Completo
```bash
backup-client restore --policy policies\policy_01.yaml --key "backups/mvp-default/20260201T190933Z/<backup_id>/backup.tar.gz.enc" --out .\restore
```

---
### 1.4 Restore Seletivo
```bash
backup-client restore --policy policies\policy_01.yaml --key "backups/mvp-default/20260201T190933Z/<backup_id>/backup.tar.gz.enc" --out .\restore_seletivo --only "*.png"
```

📌 O filtro --only atua sobre relative_path do manifest.json.

---

## 2️⃣ Agent – Orquestrador Local (intellistor-agent)

> Usado para operações oficiais, auditoria, governança e produção.
> Sempre gera logs locais e envia logs ao S3.

### 2.1 Execução Manual Governada (Agent Run)
Executa backup com logs persistidos e upload para S3.

```bash
intellistor-agent run \
  --policy policies\policy_01.yaml \
  --out out_agent \
  --env-file .\.env
```

🔎 O que acontece:
- Executa o SDK internamente
- Gera logs por policy_id
- Renomeia run_<timestamp>.log com timestamp do SDK
- Atualiza last.log
- Envia logs para:

```text
C:\ProgramData\Intellistor\Agent\logs\mvp-default\
s3://<AWS_S3_BUCKET>/logs/mvp-default/
```
---
### 2.2 Criar / Atualizar Agendamento (Windows Task Scheduler)

Cria tarefa diária baseada no cron informado.
```bash
intellistor-agent apply-schedule \
  --policy-id mvp-default \
  --cron "0 2 * * *" \
  --policy-path policies\policy_01.yaml \
  --out out_agent \
  --env-file .\.env
```

✔️ Cria:
- Task: IntellistorAgent - mvp-default
- Wrapper .cmd em:

```text
C:\ProgramData\Intellistor\Agent\run_mvp-default.cmd
```

---
### 2.3 Testar Execução do Agendamento (Manual)
Dispara a task imediatamente, sem esperar o horário.

```bash
intellistor-agent test-schedule --policy-id mvp-default
```

---
### 2.4 Verificar Status da Task

```bash
schtasks /Query /TN "IntellistorAgent - mvp-default" /V /FO LIST
```

Campos importantes:
- Horário da última execução
- Último resultado (0 = sucesso)
- Próxima execução

---
### 2.5 Remover Agendamento
```bash
intellistor-agent remove-schedule --policy-id mvp-default
```

---
### 2.6 Execução Manual via Agent (com logs + S3)
```bash
intellistor-agent run --policy policies\policy_01.yaml --out out_agent --env-file .\.env
```

O que esse comando garante (ponto importante para o comitê):
- Executa o backup real (objeto vai para o S3)
- Gera logs locais em:

```text
C:\ProgramData\Intellistor\Agent\logs\<policy_id>\
 ├── run_<timestamp>.log
 └── last.log
```

- Envia automaticamente os .log para:

```text
    s3://<AWS_S3_BUCKET>/logs/<policy_id>/
```
- Usa o mesmo .env do SDK e do Agent
- Produz evidência auditável, mesmo sem agendamento

👉 Esse é o comando oficial para execução manual auditável
(diferente do backup-client backup, que é apenas técnico).

---
## 3️⃣ Validações Operacionais e de Auditoria
### 3.1 Logs Locais
```text
dir "C:\ProgramData\Intellistor\Agent\logs\mvp-default"
type "C:\ProgramData\Intellistor\Agent\logs\mvp-default\last.log"
```

Arquivos esperados:
```text
run_20260203T215733Z.log
last.log
```
---
### 3.2 Logs no S3
```text
s3://<AWS_S3_BUCKET>/logs/mvp-default/
 ├── run_20260203T215733Z.log
 └── last.log
```

✔️ O timestamp do nome do log bate exatamente com:
- timestamp do SDK
- path do backup
- manifest
- restore

---
## 4️⃣ Regra de Ouro (Documentável)

> 🔐 Toda execução que precise de rastreabilidade, auditoria ou evidência operacional deve ser feita via intellistor-agent.

Uso direto do SDK é indicado apenas para:
- desenvolvimento
- testes locais
- troubleshooting pontual

## ✅ Conclusão Executiva
- SDK → motor técnico (backup/restore)
- Agent → governança, logs, agendamento, auditoria
- S3 → dados de backup e evidências operacionais
- Timestamp único → correlação perfeita ponta a ponta

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com.br](https://www.intellistor.com.br)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados
_Simplicidade operacional, controle e segurança._





