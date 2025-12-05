# 📑 Padrão de Status Codes da API

Este documento descreve os status codes padronizados utilizados nos endpoints das APIs da solução **Intellistor**, conforme definido na função **api_responses_utils.py**.

---
## ✅ 200 → Leitura ou atualização com sucesso
- Indica que a operação foi concluída corretamente e há retorno no corpo da resposta.
- Usado em endpoints de GET (consulta de dados) e PUT/PATCH (atualização de registros).

---
## ✅ 201 → Criação com sucesso
- Indica que um novo recurso foi criado com sucesso.
- Usado em endpoints de POST (inserção de registros).
- Retorna o recurso criado ou informações de referência (ex.: id).

---
## ✅ 202 → Requisição aceita (processamento assíncrono) 
- Indica que a requisição foi **aceita pelo servidor**, mas o processamento ainda não foi concluído.  
- Usado em operações **assíncronas**, como "esqueci minha senha" (disparo de e-mail), geração de relatórios ou tarefas em background.  
- O corpo da resposta contém apenas uma **mensagem de confirmação**, sem o resultado final da opera

---
## ✅ 204 → Exclusão com sucesso (sem corpo)
- Indica que a operação de exclusão foi concluída.
- Usado em endpoints de DELETE.
- Não retorna corpo na resposta.

---
## ⚠️ 400 → Erro de validação do input
- Indica que os dados enviados no request estão inválidos ou incompletos.
- Usado quando a entrada não passa em validações de formato, obrigatoriedade ou consistência.

---
## ⚠️ 401 → Não autenticado (Unauthorized)
- Indica que o usuário não está autenticado ou o token de acesso é inválido/expirado.
- Usado quando o cliente não fornece credenciais válidas.
- Exemplo: ausência de JWT no header Authorization.

---
## ⚠️ 403 → Acesso negado (permissões insuficientes)
- Indica que o usuário não possui as permissões necessárias para executar a operação.
- Usado em endpoints protegidos por roles ou serviços específicos.

---

## ⚠️ 404 → Recurso não encontrado
- Indica que o recurso solicitado não existe.
- Usado em consultas ou exclusões quando o identificador não corresponde a nenhum registro.

---
## ⚠️ 409 → Conflito (registro já existe)
- Indica que a operação não pode ser concluída devido a um conflito com o estado atual do recurso.
- Usado em tentativas de criação de recursos que já existem (ex.: e-mail duplicado, chave única violada).

---
## ⚠️ 422 → Erro de validação do schema (Unprocessable Entity)
- Indica que o servidor entendeu a requisição, mas os dados não seguem o formato esperado pelo schema de validação (Pydantic).
- Usado automaticamente pelo FastAPI quando os parâmetros do body, query ou path não passam nas regras definidas no modelo (BaseModel).- 
- Exemplo: campo obrigatório ausente, tipo incorreto (str em vez de int), formato inválido de e-mail, etc.

---
## ❌ 500 → Erro interno inesperado
- Indica que ocorreu um erro não tratado no servidor.
- Usado como fallback para falhas inesperadas (ex.: exceção não capturada, indisponibilidade de serviço interno).
- Não deve expor detalhes técnicos ao cliente.

---
## 👉 Padronização
Qualquer rota da API deve usar apenas esses status codes, garantindo padronização e clareza no consumo.

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com](https://www.intellistor.com)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados
