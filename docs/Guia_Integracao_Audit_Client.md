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
> Se falhar todas as tentativas de 'retry', ele retorna None e registra no arquivo de LOG interno do cliente — sem nunca impactar o usuário final.

### 4. config/settings.py

Define variáveis essenciais:
* AUTH_AUDIT_URL
* AUDIT_TIMEOUT
* Caminhos das chaves M2M (PRIVATE_KEY_PATH)
* Nome lógico do serviço (SERVICE_NAME)

📌  **Importante**:
> Esse módulo garante que behavior e endpoints sejam configuráveis.

---

## 📄 Fluxo de Retry e Logs Internos

### Retry e Registro de Logs Internos (Resiliência do Cliente)

A função _send_event_to_auth()_ implementa um mecanismo de resiliência para garantir que qualquer API cliente continue operando normalmente mesmo que a API Auth esteja temporariamente indisponível durante o envio de eventos de auditoria.

O fluxo consiste em:
1. Envio normal do evento para a Auth
2. Em caso de falha, realização automática de retries com backoff exponencial
3. Caso todas as tentativas falhem, o evento é registrado integralmente no log local para rastreabilidade
4. A aplicação segue funcionando sem bloqueios

### Política de tratamento de códigos HTTP da API Auth

Os códigos de resposta são divididos em erros recuperáveis (com retry) e erros permanentes (sem retry):

🟢 Erros recuperáveis — retry permitido

| Código  | Significado                  | Ação             |
| ------- | ---------------------------- | ---------------- |
| **500** | Erro interno da Auth         | Tentar novamente |
| **503** | Auth indisponível / overload | Tentar novamente |

🔴 Erros permanentes — retry NÃO resolve

| Código  | Significado             | Motivo                 | Ação            |
| ------- | ----------------------- | ---------------------- | --------------- |
| **401** | Token M2M inválido      | Erro de autenticação   | Não fazer retry |
| **403** | Serviço sem permissão   | Erro de autorização    | Não fazer retry |
| **404** | Endpoint não encontrado | Configuração incorreta | Não fazer retry |

### Estratégia de Retry
O módulo aplica backoff exponencial:
* 1ª tentativa → no ato
* 2ª tentativa → 1 segundo
* 3ª tentativa → 2 segundos
* 4ª tentativa → 4 segundos

Essa estratégia reduz carga no servidor e evita tempestades de requests durante quedas.

### Registro de Log Local (Fallback)

Se, após todas as tentativas, a comunicação com a API Auth permanecer indisponível, o evento não é perdido silenciosamente.
O cliente registra em log:
* O erro final
* Todas as exceções capturadas
* O payload completo que seria enviado à Auth

Exemplo do log gerado:

```csharp
[AUDIT] Todas as tentativas falharam. Evento PERDIDO.
[AUDIT] EVENTO PERDIDO (payload final): {
    "uid_user": "mock_user_key_12345",
    "event": "CREATE",
    "action": "Cluster created",
    "input_event": {...},
    "output_event": {...},
    "origem": "management"
}
``` 

📌  **Importante**:
> Isso garante rastreabilidade total mesmo sem fila ou buffer persistente em banco. Como os serviços da Intellistor utilizam bind-mount para persistir o diretório de logs fora do container, o registro do evento perdido permanece disponível mesmo que o container seja recriado, reiniciado ou substituído. Dessa forma, nenhuma informação de auditoria é perdida — o payload completo fica armazenado no log local e pode ser consultado ou reprocessado posteriormente.

### Resumo do Comportamento do Cliente em Caso de Falha

* A API cliente não trava
* O fluxo de negócio do usuário não é afetado
* Retries são executados de forma assíncrona via BackgroundTask
* Se nada funcionar, o log local preserva todas as informações
* O evento pode ser reprocessado manualmente se necessário

---

## 📤 Envio de E-mail em caso de falha do retry

O módulo **audit_client** implementa um mecanismo de alerta automático via e-mail para casos em que o evento de auditoria não puder ser registrado na API Auth após todas as tentativas de retry.

O fluxo completo funciona da seguinte forma:
1. O **audit_client** tenta enviar o evento para a API Auth.
2. Em caso de falha, o módulo aplica até 4 tentativas com backoff exponencial.
3. Se todas falharem, o evento é marcado como não entregue.
4. O payload é preservado integralmente no log local da aplicação, garantindo rastreabilidade.
5. Um e-mail é enviado automaticamente para os destinatários configurados, contendo:
    * resumo do evento
    * horário
    * identificação da origem
    * orientação técnica para verificação da API Auth

📌  **Importante**:
> Este mecanismo garante visibilidade imediata ao time responsável e evita perda de auditoria mesmo em cenários de indisponibilidade temporária.

Em caso de falha definitiva:
* Retry executado → até 4 tentativas (1s, 2s, 4s, 8s)
* Persistência local → payload salvo inteiramente em /opt/app/log/...
* Alerta → sistema dispara um e-mail automático para os administradores
* Ação recomendada → verificar saúde da API Auth e restabelecer conectividade

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

        """
        Corpo da função rota... 
        """

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

© Intellistor Solution – Todos os direitos reservados<br>
_Simplicidade operacional, controle e segurança._


