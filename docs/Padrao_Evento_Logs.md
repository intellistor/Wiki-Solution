# 📄 Padrão Oficial de Evento de LOG – Auditoria Centralizada (API Auth)
Versão 1.0 – Documento Técnico Oficial da Solução

## 🧭 1. Objetivo e Escopo

Este documento define o padrão oficial de evento de LOG auditável utilizado em toda a plataforma.
A API Auth é a única responsável por registrar eventos de auditoria no banco de dados, enquanto as demais APIs apenas geram os eventos e os enviam para o endpoint:
```bash
POST /audit/logs
```
⚠️ **Atenção**:
> Este padrão deve ser seguido por todas as APIs, serviços internos, módulos M2M e componentes de frontend que emitirem eventos de auditoria.

---
## 🎯 2. Propósito da Padronização
* Garantir consistência entre todos os serviços.
* Garantir integridade e rastreabilidade completa para auditoria corporativa e conformidade.
* Padronizar o payload oficial aceito pela API Auth.
* Facilitar análise por times de segurança, infra, desenvolvimento e FinOps.
* Suporte a investigações, troubleshooting e monitoramento unificado.

---
## 🌐 3. Fluxo da Auditoria Centralizada
1. Qualquer API ou módulo gera o evento.
2. O evento é enviado para a API Auth.
3. A API Auth valida o token M2M, normaliza o payload e registra no BD.
4. A API Auth retorna confirmação ou erro padronizado.

⚠️ **Atenção**:
> Somente a API Auth acessa o banco de auditoria. 

---

## 🏗️ 4. Estrutura Oficial do Evento
Payload aceito pela rota /audit/logs:

```json 
{
  "uid_user": "11111111-aaaa-1111-aaaa-111111111111",
  "auth_type": "JWT",
  "event": "LOGIN",
  "action": "User authenticated successfully",
  "origin": "Intellistor Platform",
  "input_event": {
    "endpoint": "/auth/login",
    "ip": "10.0.0.10",
    "body": {
        "user": "machado", 
        "service": "admin"
  },
  "output_event": {
    "code": 200,
    "status": "success"
  }
}
```

### ❗ **Importante**:
Cada evento deve representar uma ação única, com contexto suficiente para identificar:
* Quem fez
* O que fez
* Onde fez
* Resultado da ação
* Origem e impacto da operação

---
## 🧩 5. Campos Oficiais do Evento de LOG
### 5.1 uid_user
| Tipo            | Obrigatório | Descrição                                     |
| --------------- | ----------- | --------------------------------------------- |
| `string (UUID)` | Sim         | Identificador único do usuário ou agente M2M. |

### 5.2 auth_type
| Valor | Descrição                                              |
| ----- | ------------------------------------------------------ |
| `JWT` | Evento gerado por autenticação via token do usuário.   |
| `M2M` | Evento gerado por integração entre APIs via chave M2M. |

### 5.3 event
Representa o tipo da operação.
Valores aceitos:
| Evento                       | Aplicação                                       |
| ---------------------------- | ----------------------------------------------- |
| `LOGIN`                      | Autenticação do usuário                         |
| `LOGOUT`                     | Encerramento de sessão                          |
| `TOKEN_REFRESH`              | Renovação de token                              |
| `CREATE`, `UPDATE`, `DELETE` | CRUD padronizado                                |
| `INTEGRATION`                | Chamadas API ↔ API                              |
| `AUDIT`                      | Operações críticas                              |
| `CONFIG`                     | Ações de configuração                           |
| `OBJECT`                     | Ações relacionadas a objetos (S3, buckets etc.) |

### 5.4 action
Descrição textual da ação executada.
Exemplo:
* "User authenticated successfully"
* "Storage created"
* "Policy updated"

### 5.5 origem
Identifica de qual API ou aplicação o evento de LOG foi gerado.
Esse campo é fundamental para rastreabilidade, filtros de consulta e correlação entre módulos da plataforma.
| Tipo               | Obrigatório | Descrição                                                              |
| ------------------ | ----------- | ---------------------------------------------------------------------- |
| `string (VARCHAR)` | Sim         | Identificador da origem emissora do evento (API ou aplicação cliente). |

**Valores recomendados**
| Valor        | Uso previsto                                         |
| ------------ | ---------------------------------------------------- |
| `auth`       | Eventos gerados pela API Auth                        |
| `management` | Eventos gerados pela API Management                  |
| `licensing`  | Eventos gerados pela API Licensing                   |
| `integrator` | Eventos gerados pela API Integrator                  |
| `frontend`   | Eventos originados diretamente Intellistor Platform  |
| `scheduler`  | Tarefas agendadas / jobs em background               |
| `external`   | Eventos provenientes de sistemas externos (opcional) |

Regras:
* Deve sempre refletir a origem lógica do evento, não apenas o host de rede.
* Em chamadas M2M, o valor de origem deve ser coerente com a identidade da API associada à chave pública utilizada.
* Pode ser usado como critério de filtro em consultas de auditoria e dashboards.

### 5.6 input_event
Objeto contendo os dados recebidos pela operação.
| Campo             | Descrição                              |
| ----------------- | -------------------------------------- |
| `endpoint`        | Rota ou processo que originou o evento |
| `ip`              | IP de origem                           |
| `body` (opcional) | Payload recebido pela API              |

### 5.7 output_event
Objeto contendo o resultado do processamento.
| Campo               | Descrição                          |
| ------------------- | ---------------------------------- |
| `code`              | Código HTTP retornado              |
| `status`            | `"success"`, `"error"`, `"failed"` |
| `detail` (opcional) | Mensagem técnica                   |

---
## 🔐 6. Regras de Severidade (derivadas de output_event.status)

| Status    | Significado                  |
| --------- | ---------------------------- |
| `success` | Operação executada com êxito |
| `failed`  | Execução incompleta          |
| `error`   | Falha crítica                |

A API Auth transforma esse status interno nos campos equivalentes que são persistidos no banco.

---
## 👤 7. Usuário (Campo uid_user)
| Origem            | Regra                    |
| ----------------- | ------------------------ |
| **Usuário final** | UID extraído do JWT      |
| **API cliente**   | UID identificado via M2M |
 
 ---
## 🕒 8. Data do Evento
Mesmo que o payload contenha timestamp, a API Auth ignora e gera o horário real do processamento.

⚠️ **Atenção**:
> A data oficial registrada é sempre a gerada internamente pela Auth.

---
## ⚙️ 9. Tratamento Interno dos Campos (Auth)

A API Auth realiza:
* Normalização do evento
* Serialização JSON segura
* Remoção de campos inválidos
* Preenchimento de valores padrão
* Registro de data/hora real
* Fallback seguro em caso de erro
* Persistência em tabela audit_logs

---
## 📌 10. Exemplo Completo de Evento Validado
Resultado final (já normalizado pela Auth):
```json
{
  "uid_user": "11111111-aaaa-1111-aaaa-111111111111",
  "auth_type": "JWT",
  "event": "LOGIN",
  "action": "User authenticated successfully",
  "input_event": {
    "endpoint": "/auth/login",
    "ip": "10.0.0.10",
    "body": {
        "user": "machado", 
        "service": "admin"
  },
  "output_event": {
    "code": 200,
    "status": "success"
  }
  "data_evento": "2025-12-05T18:00:00Z",
  "origin": "Intellistor Platform"
}
```

---
## 📘 11. Boas Práticas
* Enviar somente o necessário em input_event e output_event.
* Nunca enviar dados sensíveis em texto puro.
* Registrar sempre uma única ação por evento.
* Usar M2M em todas as integrações entre APIs.
* Não manipular timestamp — Auth gera internamente

---
## 🧪 12. Erros Comuns de Implementação
* Enviar payload sem uid_user.
* Enviar auth_type incorreto.
* input_event fora do padrão objeto.
* output_event.code não numérico.

---
## 📄 13. Governança e Atualizações

⚠️ **Atenção**:
> Alterações neste padrão devem ser validadas pela equipe responsável e refletidas imediatamente na API Auth.
> Mudanças sem aprovação podem causar inconsistência no banco de auditoria.

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com.br](https://www.intellistor.com.br)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados


