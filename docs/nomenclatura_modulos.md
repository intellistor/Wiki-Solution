# 📘 Padões de Nomenclatura - Módulos

Este guia define o padrão oficial de nomenclatura para módulos Python no projeto **SM/Intellistor**, promovendo consistência, clareza e escalabilidade.

---

## 🧭 Índice

- [Formato Geral](#formato-geral)
- [Definição de Sufixos](#definição-de-sufixos)
  - [`_service`](#service)
  - [`_client`](#client)
  - [`_tasks`](#tasks)
  - [`_utils`](#utils)
  - [`_repository`](#repository)
  - [`_schema`](#schema)
- [Quadro de Decisão](#quadro-de-decisão)
- [Boas Práticas](#boas-práticas)

---

## 🧩 Formato Geral

```plaintext
<domínio>[_<tecnologia>]_<sufixo>
````
domínio: área funcional (ex.: lun, auth, permission, storage_pool)
tecnologia: opcional, usado quando há dependência clara (ex.: huawei, smtp, s3)
sufixo: define o tipo de componente

---
## 🏷️ Definição de Sufixos
⚙️ _service

Módulo que implementa lógica de negócio e orquestra processos. Reúne regras, validações e chamadas a outras camadas (_client, _repository, _utils).

Exemplos:
````plaintext
    permission_service.py
    jwt_auth_service.py
    provisioning_service.py
````

---
## 🌐 _client

Módulo que integra com sistemas externos (APIs, SDKs, protocolos, fabricantes). Contém chamadas diretas sem regra de negócio.

Exemplos:

````plaintext
    email_smtp_client.py
    s3_bucket_client.py
    huawei_dorado_client.py
````
---
## ⏱️ _tasks

Módulo que implementa tarefas assíncronas ou agendadas, executadas por workers (Celery, RQ, APScheduler).

Exemplos:
````plaintext
    capacity_monitoring_tasks.py
    report_generation_tasks.py
````

---
## 🛠️ _utils

Módulo com funções auxiliares genéricas e reutilizáveis, sem lógica de negócio.

Exemplos:
````plaintext
    date_utils.py
    retry_utils.py
````

---
🗄️ _repository

Módulo responsável por persistência e acesso a banco de dados (DAO). Implementa CRUD e queries específicas.

Exemplos:
````plaintext
    event_log_repository.py
    user_repository.py
````

---
📦 _schema

Módulo que define modelos de dados e validações (Pydantic) para entrada/saída em APIs.

Exemplos:
````plaintext
    lun_schema.py
    auth_schema.py
````

---
## 🧮 Quadro de Decisão – Qual sufixo usar?

| ❓ Pergunta                                                   | ✅ Resposta | 🏷️ Sufixo     | 📁 Exemplo                    |
|---------------------------------------------------------------|-------------|---------------|-------------------------------|
| Implementa regras de negócio ou orquestra fluxos?            | Sim         | `_service`    | `permission_service.py`       |
| Conecta-se a sistema externo (API, SDK, protocolo)?          | Sim         | `_client`     | `email_smtp_client.py`        |
| É processo assíncrono ou agendado?                           | Sim         | `_tasks`      | `capacity_monitoring_tasks.py`|
| Contém funções auxiliares genéricas?                         | Sim         | `_utils`      | `date_utils.py`               |
| Faz acesso direto ao banco de dados?                         | Sim         | `_repository` | `event_log_repository.py`     |
| Define modelos de dados/validação (API)?                     | Sim         | `_schema`     | `auth_schema.py`              |


---
## ✅ Boas Práticas

Combine domínio + tecnologia + sufixo se necessário (ex.: lun_huawei_client.py, email_smtp_client.py)

- Evite nomes genéricos como common.py, helpers.py; seja explícito no domínio
- Se o módulo se encaixar em dois papéis, divida em dois arquivos (ex.: um _client e um _service)
- Use sempre inglês, snake_case, nomes **curtos e claros`

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com](https://www.intellistor.com)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados


