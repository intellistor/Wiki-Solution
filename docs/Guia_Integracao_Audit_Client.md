
## 📦 Audit Client Module – Guia de Implementação para Envio de Eventos à API Auth

```markdown
audit_client
├── __init__.py
├── builder.py
├── config
│   ├── __init__.py
│   └── settings.py
├── sender.py
└── tasks.py
``` 
Este módulo descreve toda a lógica necessária para que uma API cliente (Management, Licensing, Integrator etc.) gere e envie eventos de auditoria para a API Auth.

---
## 📁 Módulos e Responsabilidades

### 1. builder
```python
builder.py
➜ Função: build_audit_event()
```
Responsável por construir o payload oficial que será enviado para a API Auth. 
Essa função:
* Aplica os campos obrigatórios
* Normaliza as estruturas input/output
* Define corretamente o campo origem
* Garante consistência entre todas as APIs clientes

Exemplo de uso na rota::
```python
event = build_audit_event(
    user_key=identity,
    event="CREATE",
    action="Cluster created",
    input_event={"endpoint": "/cluster", "body": body.dict()},
    output_event={"code": 200, "status": "success"},
)
```
📌 **Observação importante**:<br>
> ➡️ A chave **user_key** enviado pelo cliente é o **identity** que será usado para recuperar o **uid_user**<br>
> ➡️ A API Auth recupera o **uid_user** real antes de salvar no banco.

### 2. tasks
```python
tasks.py
➜ Função: audit_event()
```

Responsável por:
* Receber o evento construído
* Acionar o envio em BackgroundTask
* Garantir que a rota principal não espere a auditoria
* Evitar que falhas de auditoria afetem o fluxo da API cliente

Exemplo de uso na rota:
```python
audit_event(background_tasks, event)
```
📌  **Importante**:
> Isso dispara o envio assíncrono, sem travar o endpoint da API Client.

### 3. sender
Contém as duas funções críticas:

```python
sender.py
➜ Função: sign_request()
➜ Função: send_event_to_auth()
```

✔️ sign_request()

Gera os **headers de autenticação M2M** que comprovam para a Auth:
* Quem está enviando o evento
* Que a API emissora é legítima
* Que o token não foi forjado

Esta função:
* Invoca a função generate_m2m_token(), que gera o Token M2M
* Preenche os headers obrigatórios:
    ```body
    Authorization: Bearer <token>
    X-M2M-Service: management-api
    Content-Type: application/json
    ```

✔️ send_event_to_auth()

Envia o evento para a API Auth via requests.post():
* Usa timeout configurado
* Usa headers assinados
* Não deixa erro de rede quebrar o fluxo da API Client

⚠️ **Atenção**:
> Se falhar, ele retorna None e registra log interno — sem nunca impactar o usuário final.

### 4. config/settings.py

Define variáveis essenciais:
* AUTH_AUDIT_URL
* AUDIT_TIMEOUT
* Caminhos das chaves M2M (PRIVATE_KEY_PATH)
* Nome lógico do serviço (SERVICE_NAME)

📌  **Importante**:
> Esse módulo garante que behavior e endpoints sejam configuráveis.

---
## 🧠 Fluxo completo do módulo audit_client
```scss
1) Rota executa regra de negócio
2) builder.build_audit_event() monta o evento
3) tasks.audit_event() coloca na fila de background
4) sender.sign_request() gera token M2M
5) sender.send_event_to_auth() envia o evento para Auth
6) API Auth valida M2M, resolve user_key → uid_user e grava auditoria
```

⚠️ **Atenção**:
> Este é o **padrão oficial** de auditoria da Intellistor Solution.

---
## ✅ Rota completa com auditoria integrada (exemplo) 
```python
@api.post(
    "/",
    summary="Register new cluster",
    status_code=201,
    responses=api_responses("cluster", codes=[201, 401, 409, 417, 422, 500]),
    dependencies=_LICENSE_DEPS + _JWT_PERMISSION,
)
def create_cluster(
        body: ClusterBase,
        background_tasks: BackgroundTasks
):
    try:
        identity = jwt_identity.get()
        logger.info(
            f"[CLUSTER] Solicitação de registro do cluster '{body.name}' realizada por {identity}"
        )

        """Converte o body em dict"""
        dados = body.dict()

        """Insere na base SQL"""
        result, status_code = TabelaCluster.insert_ic_cluster(dados)
        if status_code != 200:
            return default_error(
                error_msg=translate_msg("msg.error.general", msg_error=result),
                code_erro=status_code,
            )

        """Registra o evento de LOG de auditoria (integração com API AUTH)"""
        event = build_audit_event(
            user_key=identity,
            event="CREATE",
            action="Cluster created",
            input_event={"endpoint": "/create_cluster", "body": body.dict()},
            output_event={"code": f"{status_code}", "status": "success"}
        )
        audit_event(background_tasks, event)

        """Constroi o output de sucesso"""
        output = msg_success_api("api.register.created", data=dados)
        return default_return(output, 201)

    except Exception as e:
        msg_erro = (
            f"[CLUSTER] "
            f"{translate_msg('endpoint.error', name=body.name, msg_erro=f'{e}')}"
        )
        return default_error(error_msg=msg_erro, code_erro=500)

```
---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com.br](https://www.intellistor.com.br)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados


