# 📦 Criação e Distribuição de SDKs Intellistor (Modelo .whl On-Premise)

Este documento define o padrão oficial da Intellistor para criação, versionamento, build e distribuição de SDKs utilizados pelas APIs da plataforma.

O objetivo é fornecer um caminho simples, padronizado e 100% compatível com ambientes on-premise, sem dependência de GitHub, PyPI ou internet externa.

---
## 🎯 1. Objetivo
Os SDKs da Intellistor (ex.: intellistor_audit_sdk) devem ser instalados pelas APIs da solução via pacote .whl local, garantindo:
* Reprodutibilidade
* Operação offline
* Segurança em ambientes restritos
* Padronização entre microserviços
* Facilidade de evolução e versionamento

---
## 🧱 2. Estrutura mínima de um SDK
Todo SDK deve seguir esta estrutura:

```markdown
intellistor_nome_sdk/
├── intellistor_nome/
│   ├── __init__.py
│   ├── modulo1.py
│   ├── modulo2.py
│   └── ...
├── README.md
└── pyproject.toml
```

⚠️**Atenção:***
> Nome do SDK (pacote distribuído): intellistor-nome-sdk
> Nome do pacote Python: intellistor_nome

---
## ⚙️ 3. Arquivo obrigatório: **pyproject.toml**
Exemplo funcional:

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "intellistor-audit-sdk"
version = "1.0.0"
description = "Intellistor Audit Client SDK for centralized audit logging"
readme = "README.md"
requires-python = ">=3.10"
authors = [
  { name = "Intellistor Tecnologia", email = "support@intellistor.com.br" }
]
dependencies = [
  "requests>=2.31.0",
  "PyJWT>=2.8.0",
  "cryptography>=42.0.0",
  "loguru>=0.7.2",
]

[tool.setuptools]
packages = ["intellistor_audit"]
```
📌  **Regras do projeto:**
> Sempre usar versionamento semântico (ex.: 1.0.0, 1.1.0, 2.0.0)
> O nome do pacote interno (packages) deve bater com o diretório Python

---
## 🛠️ 4. Instalando a ferramenta de build
Rodar apenas uma vez por ambiente:

```bash
pip install build
``` 

---
## 🏗️ 5. Como gerar o pacote .whl
Na raiz do SDK:
```markdown
intellistor_audit_sdk/
├── pyproject.toml
├── README.md
└── intellistor_audit/
```
Execute:
```bash
python -m build
```

Isso cria:
```markdown
intellistor_audit_sdk/
├── pdist/
     ├── intellistor_audit_sdk-1.0.0-py3-none-any.whl
     └── intellistor_audit_sdk-1.0.0.tar.gz
├── pyproject.toml
├── README.md
└── intellistor_audit/
```

O arquivo usado pelas APIs é:
```bash
intellistor_audit_sdk-1.0.0-py3-none-any.whl
```

---
## 📥 6. Como distribuir o SDK no instalador on-premise

Recomendamos esta estrutura:
```markdown
api-intellistor-management/
├── packages/
│   └── intellistor_audit_sdk-1.0.0-py3-none-any.whl
├── requirements.txt
└── docker-compose.yml
```

No requirements.txt:
```markdown
packages/intellistor_audit_sdk-1.0.0-py3-none-any.whl
```

📌  **Por quê isso é essencial?**

> ✔️ Totalmente offline<br>
> ✔️ Build determinístico<br>
> ✔️ Reprodutível entre ambientes<br>
> ✔️ Clientes não precisam acessar GitHub ou outro repositório qualquer

---
## 🔄 7. Como atualizar a versão do SDK

Abra o SDK e altere no pyproject.toml:
```bash
version = "1.1.0"
```
Gere o novo pacote:
```bash
python -m build
```
Substitua no repo do serviço:
```bash
packages/intellistor_audit_sdk-1.1.0-py3-none-any.whl
```
Atualize o requirements.txt:
```bash
packages/intellistor_audit_sdk-1.1.0-py3-none-any.whl
```
Rebuild da imagem Docker:
```bash
docker-compose build
````

---
## 🔒 8. Boas práticas corporativas Intellistor
✔️ Nunca usar Git+HTTPS no requirements para SDK interno.<br>
  Ambiente do cliente pode não ter:<br>
  > internet<br>
  > acesso ao GitHub<br>
  > proxy liberado para HTTPS externo<br>

✔️ Usar sempre .whl local<br>
✔️ Versões fixas<br>
✔️ Alterações documentadas no CHANGELOG<br>
✔️ SDKs versionados seguindo SemVer<br>

---
## 📄 9. Checklist final para criar um SDK Intellistor

| Item                                       | OK |
| ------------------------------------------ | -- |
| Estrutura do pacote criada                 | ⬜  |
| `pyproject.toml` configurado               | ⬜  |
| `python -m build` gerou o `.whl`           | ⬜  |
| `.whl` copiado para `packages/` do serviço | ⬜  |
| `requirements.txt` atualizado              | ⬜  |
| Testes manuais concluídos                  | ⬜  |
| Docker rebuild OK                          | ⬜  |

---
## 📌 Conclusão
Este documento formaliza o padrão da Intellistor para desenvolvimento e distribuição de SDKs, garantindo:
* Confiabilidade
* Reprodutibilidade
* Conformidade com ambientes on-premise
* Padronização entre equipes e microserviços
* Segurança corporativa

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com.br](https://www.intellistor.com.br)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados

