# 📦 Intellistor Backup Solution – Guia Rápido (MVP)

**Data:** 01/02/2026<br>
**Escopo:** MVP funcional do Intellistor Backup Solution (SDK + Agent + Agendamento Windows)<br>
**Objetivo:** Demonstrar um MVP funcional de Backup/Restore com criptografia, manifest, agendamento automático no Windows (Task Scheduler) e upload do objeto de backup, juntamente com os logs organizados por *policy_id*, para o S3.

---
## Pré-requisitos

<img width="1573" height="730" alt="image" src="https://github.com/user-attachments/assets/89fbf28a-a635-4b02-a5b4-f15c2dbb29ca" />

### Ambiente

* Sistema operacional: Windows (MVP)
* Python: 3.11+
* Acesso à AWS S3
* SDK e Agent instalados no mesmo ambiente virtual (`.venv`)

### Arquivo `.env`

O SDK e o Agent dependem do `.env`.
O Task Scheduler **não herda variáveis do shell**, portanto o `.env` deve ser informado explicitamente ao Agent.

Exemplo mínimo:

```env
AWS_ACCESS_KEY_ID=xxxxxxxx
AWS_SECRET_ACCESS_KEY=xxxxxxxx
AWS_DEFAULT_REGION=us-east-1
AWS_S3_BUCKET=intellistor-prd
```

---


## 0️⃣ Criação da Policy

A *policy* define **o que será protegido**, **quando** e **como**.

```bash
backup-client policy-create \
  --out policies/policy_02.yaml \
  --paths "C:\workload\git_lab\docs\imagens" \
  --schedule "0 2 * * *"
```
**Resultado esperado:**

```json
{ "created": "policies/policy_01.yaml" }
```

**Notas:**
- `paths` → diretórios incluídos no backup  
- `schedule` → referência de agendamento (cron).  
  > No MVP, é apenas informativo (scheduler entra depois).
- Caso não informado:
  - `paths` assume o diretório atual
  - `schedule` assume `0 2 * * *`

---

## 1️⃣ Execução do Backup

Executa o backup conforme a policy definida.

```bash
backup-client backup \
  --policy policies/policy_02.yaml \
  --out .\out
```

### Resultado
- Criação do backup criptografado (`.tar.gz.enc`)
- Upload para o bucket S3
- Geração do `manifest.json`
- Retorno em JSON contendo, entre outros campos:
  - `backup_id`
  - `timestamp`
  - `s3.key_backup`
  - `s3.key_manifest`
  - `cli_restore` → comando pronto para restore (copy/paste)

Exemplo de retorno parcial:
```json
{
  "backup_id": "9e19eaa1-0bdb-428a-b176-b288670e68ea",
  "timestamp": "20260131T174311Z",
  "policy": "mvp-default",
  "cli_restore": "backup-client restore --policy policies/policy_02.yaml --key 'backups/.../backup.tar.gz.enc' --out .\\restore"
}
```

---

## 2️⃣ Restore

O restore pode ser **completo** ou **seletivo**.

---

### 2.1 🔄 Restore Geral (Completo)

Restaura **todo o conteúdo** do backup.

```bash
backup-client restore \
  --policy policies/policy_02.yaml \
  --key "backups/mvp-default/20260131T162926Z/77a270fd-c77d-4dc3-bc0c-78b52c80515c/backup.tar.gz.enc" \
  --out .\restore
```

**O que acontece:**
- Download do backup criptografado
- Download do `manifest.json`
- Descriptografia
- Extração completa
- Verificação de integridade (SHA-256)

---

### 2.2 🎯 Restore Seletivo (`--only`)

Restaura **apenas arquivos compatíveis com o filtro informado**.

> O filtro `--only` é aplicado **exclusivamente** sobre o campo  
> `relative_path` do `manifest.json`.

```bash
backup-client restore \
  --policy policies/policy_02.yaml \
  --key "backups/mvp-default/20260131T162926Z/77a270fd-c77d-4dc3-bc0c-78b52c80515c/backup.tar.gz.enc" \
  --out .\restore_seletivo \
  --only "Cópia*"
```

#### Exemplos de filtros válidos
```text
"Diagrama_Solução.png"
"*.png"
"docs/*"
"Cópia*"
```

**Resultado:**
- Apenas os arquivos compatíveis são restaurados
- Integridade verificada somente para os arquivos extraídos
- Retorno indicando:
  - quantidade encontrada
  - quantidade restaurada
  - status da integridade

---

## 📌 Observações Importantes

- O parâmetro `--key` **sempre aponta para**:
  ```text
  backup.tar.gz.enc
  ```
  O `manifest.json` é derivado automaticamente.
- O restore não sobrescreve arquivos fora do diretório `--out`.
- Criptografia: **AES-256-GCM client-side**
- Formato: **tar + gzip**, com restore controlado.

---

## ✅ Fluxo Recomendado

1. Criar a policy  
2. Executar o backup  
3. Copiar o `cli_restore` retornado  
4. (Opcional) Ajustar `--only` para restore seletivo  

---
## ▶️ Video 

[Video do MVP](#)

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com.br](https://www.intellistor.com.br)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados
_Simplicidade operacional, controle e segurança._
