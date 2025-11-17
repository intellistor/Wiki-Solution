# Intellistor Platform

## 1️⃣ Arquitetura Geral da Solução

A Intellistor Platform é uma solução modular e corporativa composta por:

### **APIs**

-   **API AUTH**\
    Autenticação, Autorização, Identidade (AuthN/AuthZ/Identity).\
    Integração LDAP, suporte M2M, gestão de usuários, permissões,
    logins, auditoria.

-   **API LICENSING**\
    Servidor de licenças com MySQL, emissão e validação de `.lic`, CRL,
    Ed25519.

-   **API LICENSING CLIENT**\
    Cliente automático que registra chaves no AUTH via M2M.

-   **API INTEGRATOR**\
    Automação multivendor (Huawei, NetApp, Hitachi).\
    Fluxos: LUN, NFS, CIFS, Discovery, Snapshots, Zones, Predict.

-   **API MANAGEMENT**\
    Gerencia ICs (Infraestruturas Críticas).\
    Funciona como uma mini-CMDB da solução.

-   **Frontend React -- Intellistor Platform**\
    Painéis, automações, gestão de ICs, perfil financeiro/FinOps.

------------------------------------------------------------------------

## 2️⃣ Padrões Oficiais da Solução

### **Docstrings Slim Comercial**

-   Bilíngue PT-BR/EN
-   Estrutura fixa com título HTML `<h2>`
-   Parâmetros, Retorno, Links oficiais

### **Modelos oficiais**

-   POST → create_storages
-   GET → get_storages
-   PUT → update_storages
-   PUT → status_storage (activate/deactivate)
-   DELETE → Exclusão lógica is_active=0
-   Todos com:\
    `default_return`, `default_error`, `msg_success_api`, logs
    corporativos, permissões.

------------------------------------------------------------------------

## 3️⃣ M2M -- Machine-to-Machine

Módulo completo: - Geração automática de chaves Ed25519 (X.509 PEM) -
Registro automático no AUTH - Diretório `data/m2m_keys/` - Rotas:
`POST /register_public_key` e `GET /register_public_key`

------------------------------------------------------------------------

## 4️⃣ Integração multivendor

-   Módulo de criação de NFS e CIFS para OceanStor Dorado v6.1.8.
-   Fluxo completo baseado em documentação Huawei.
-   Tag **nfs/cifs** mantém todo histórico.

------------------------------------------------------------------------

## 5️⃣ Licenciamento

-   Ed25519 Assinaturas digitais\
-   CRL\
-   Licensing Client automático\
-   Emissão `.lic`, permissões avançadas\
-   Controle centralizado no AUTH

------------------------------------------------------------------------

## 🔥 Conclusão

A Intellistor Platform contempla:

-   Arquitetura completa\
-   M2M integrado\
-   Autenticação corporativa\
-   Management como CMDB\
-   Automação multivendor\
-   Padrões oficiais de rotas\
-   SQL unificado\
-   Estratégia FinOps

A plataforma está sólida, padronizada e pronta para expansão para novos
módulos.

---

## ✉️ Contato

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os responsáveis pelo projeto:

- **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

---

