# 📧 **Envio de Email SDK**

SDK oficial para envio de e-mails HTML padronizados pelas APIs e serviços internos da Intellistor Platform.

Este guia é direcionado ao desenvolvedor que vai consumir o SDK, utilizando o arquivo .whl disponibilizado dentro da pasta packages/ da solução.

---
## 📦 **Instalação em Ambientes On-Premise (pasta packages/)**
Em ambientes corporativos **on-premise** — especialmente quando a solução Intellistor é instalada em datacenters do cliente — é comum trabalhar sem acesso direto à internet.
A Intellistor distribui suas SDKs em formato wheel (.whl) dentro da pasta:
```bash
packages/
```

Antes de usar a SDK, confirme que o arquivo está presente:
```pgsql
packages/
└── intellistor_email_sdk-<versao>-py3-none-any.whl
```

---
## ✅ **Instalação**

No terminal (dentro do ambiente virtual da aplicação):
```bash
pip install ./packages/intellistor_email_sdk-*.whl --force-reinstall
```
Ou especificando a versão:
```bash
pip install ./packages/intellistor_email_sdk-1.0.0-py3-none-any.whl --force-reinstall
```

---
## 📂 **Estrutura recomendada do repositório cliente**
```pgsql
intellistor_solution/
├── api_auth/
├── api_management/
├── api_integrator/
├── packages/
│   ├── intellistor_audit_sdk-1.0.0-py3-none-any.whl
│   ├── intellistor_m2m_sdk-1.0.0-py3-none-any.whl
│   └── intellistor_email_sdk-1.0.0-py3-none-any.whl   ← envio de e-mail
├── docker-compose.yml
└── README.md
```

⚠️ **Atenção:**
> Esse diretório armazena todos os pacotes .whl necessários para instalação offline.

---
## 🚀 **Uso básico do SDK**

```python
from intellistor_email.email_smtp_client import send_email

send_email(
    body="<p>Seu processamento foi concluído com sucesso.</p>",
    subject="Processamento Concluído",
    email_to="usuario@empresa.com"
)
```

---
## 🧩 **Uso com múltiplos destinatários e BCC**
```python
from intellistor_email.email_smtp_client import send_email

send_email(
    body="<p>Nova atualização disponível.</p>",
    subject="Atualização do Sistema",
    email_to=["devops@empresa.com", "infra@empresa.com"],
    email_bcc=["gestao@empresa.com", "auditoria@empresa.com"]
)
```

---
## 📎 **Envio com Anexo**
```python
from intellistor_email.email_smtp_client import send_email

send_email(
    body="<p>Segue o relatório solicitado.</p>",
    subject="Relatório Mensal",
    email_to="cliente@empresa.com",
    filename="/opt/relatorios/mensal.pdf"
)
```

---
## 🎨 **Envio usando HTML dinâmico**

Você pode construir o corpo da mensagem dinamicamente e enviar:
```python
from intellistor_email.email_smtp_client import send_email

html = f"""
<h2>Olá, Renato!</h2>
<p>Seu ticket #{1234} foi atualizado.</p>
"""

send_email(
    body=html,
    subject="Atualização de Ticket",
    email_to="renato@empresa.com"
)
```

---
## 👉  **Exemplo completo utilizado em APIs Intellistor**
```python
from fastapi import APIRouter
from intellistor_email.email_smtp_client import send_email

router = APIRouter()

@router.post("/notify-user")
def notify(email: str):
    body = """
    <h2>Notificação da Intellistor Platform</h2>
    <p>Seu ambiente foi provisionado com sucesso.</p>
    """

    ok = send_email(
        body=body,
        subject="[Intellistor] Ambiente Provisionado",
        email_to=email
    )

    return {"sent": ok}
```

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com.br](https://www.intellistor.com.br)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados












