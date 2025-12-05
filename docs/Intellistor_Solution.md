# 🏛️ 1. Visão Geral

A **Intellistor Solution** adota um modelo de licenciamento híbrido, oferecendo flexibilidade tanto para organizações que priorizam investimentos de capital (CAPEX) quanto para aquelas que operam com despesas recorrentes (OPEX). Essa abordagem foi desenvolvida para atender às demandas estratégicas de grandes corporações que exigem máxima eficiência na gestão de sua infraestrutura de storage e backup.

---
<img width="1413" height="572" alt="image" src="https://github.com/user-attachments/assets/ea8f734a-ec56-422f-9fd0-d1df6bb79ad3" />

---
## 🏗️ Ambiente da Peta (Intranet / Control Center / Licenciamento):
> Autoridade central para emissão, supervisão e revogação de licenças.


## 🏗️ Ambiente do Cliente (On-premise):
> Solução instalada localmente, operando em modo seguro e validando licenças através do Licensing Client.


---
A arquitetura da **Intellistor Solution** incorpora princípios de **segurança zero-trust, com autenticação avançada via assinaturas digitais Ed25519 e gerenciamento de certificados por meio de CRL distribuída em S3**. Os módulos independentes são ativados exclusivamente após aprovação da licença, garantindo confiabilidade, escalabilidade e **proteção integral em ambientes M2M**.

🔐 **Princípios de segurança Zero-Trust**
1. Zero-Trust é um modelo de segurança que parte do princípio de não confiar em ninguém por padrão, nem dentro nem fora da rede.
2. Cada acesso ou ação precisa ser verificado e autenticado continuamente.
3. Isso reduz riscos, porque mesmo usuários internos precisam provar sua identidade e autorização.

✍️ **Assinaturas digitais Ed25519**
1. Ed25519 é um algoritmo moderno de criptografia baseado em curvas elípticas.
2. Ele é usado para criar assinaturas digitais que garantem:
    Autenticidade: confirma que o conteúdo veio de quem diz ter enviado.
    Integridade: garante que o conteúdo não foi alterado.
3. É rápido e considerado altamente seguro.

📜 **CRL distribuída via S3**
1. CRL (Certificate Revocation List) é uma lista de certificados digitais que foram revogados (não são mais válidos).
2. Distribuir essa lista via Amazon S3 significa que ela fica armazenada e acessível de forma escalável e confiável na nuvem.
3. Isso garante que sistemas possam verificar rapidamente se um certificado ainda é válido ou foi cancelado.

🧩 **Módulos independentes com licença**
1. A arquitetura é composta por módulos independentes (partes separadas do sistema).
2. Cada módulo só funciona após aprovação da licença, ou seja:
    O sistema verifica se você tem permissão para usar aquele módulo.
    Isso impede uso não autorizado e garante controle comercial e técnico.

🔑 **Solução M2M**
1. A comunicação **Machine-to-Machine (M2M)** é protegida por um sistema baseado em pares de chaves criptográficas.
2. Cada dispositivo ou serviço possui uma chave privada e uma chave pública associada.
3. O processo garante:
   * _Autenticação forte_: apenas dispositivos com chaves válidas podem se comunicar.
   * _Confidencialidade_: dados são criptografados de forma que apenas o destinatário autorizado consiga decifrar.
   * _Integridade_: qualquer alteração indevida nas mensagens é detectada.
4. Essa abordagem elimina dependência de tokens temporários e assegura que toda interação M2M seja autenticada e confiável, alinhada ao modelo de licenciamento híbrido.

---

## 🟦 2. Ambiente da Peta (Intranet) – Autoridade Central de Licenciamento

O ambiente da Peta é responsável por licenciar, supervisionar e gerenciar todas as instâncias da **Intellistor Platform** instaladas nos clientes.

### Componentes

📦 **2.1 Intellistor Control Center (Intranet)**

Painel interno da Petacorp onde equipes autorizadas:
  * geram licenças (.lic)
  * acompanham consumo e incidentes
  * revogam licenças
  * atualizam CRLs
  * gerenciam chaves públicas

---
📦 **2.2 API Licensing (Peta)**

Serviço oficial de controle de licenciamento:
  * gera licenças Ed25519 assinadas
  * atualiza crl.json
  * publica chaves públicas para verificação
  * registra histórico e telemetria de uso

---
📦 **2.3 Banco de Dados Central**

Armazena:
  * licenças emitidas
  * clientes, instâncias e módulos ativos
  * logs de supervisão
  * status de ativação/revogação

---
🪣 **2.4 Repositório S3 (AWS)**

Ponto de distribuição global para os clientes:
  * crl.json (Certificate Revocation List)
  * chave pública Ed25519 da Peta

O ambiente do cliente acessa exclusivamente o S3.
Não existe comunicação direta do cliente com a intranet da Peta.

---
## 🟩 3. Ambiente do Cliente – Execução da Plataforma

O ambiente do cliente é totalmente **on-premise**, com acesso externo apenas ao AWS S3 para obter CRL.

### Componentes
📦 **3.1 API Licensing Client**

É o agente local de licenciamento da plataforma. Responsável por:
  * baixar crl.json e chave pública do S3
  * validar o arquivo .lic
  * verificar assinatura, prazo e módulos
  * ativar/desativar funcionalidades
  * informar resultado ao núcleo da plataforma

⚠️ **Atenção**:
>  É o guardião da licença.

---
📦 **3.2 API Auth (Login & Permission)**

Controla:
  * autenticação via LDAP (corporativo)
  * fallback via banco local (credenciais da plataforma)
  * autorização baseada em permissões
  * emissão de tokens JWT

---
📦 **3.3 Intellistor Platform (Core)**

Interface com o cliente. Recebe a validação da licença e habilita módulos:
  * Storage
  * Backup
  * Analytics
  * Management
  * Módulos adicionais (API XXX)

---
📦 **3.4 API Management CMDB da SAN**

Módulo principal para gestão:
  * inventário
  * ICs
  * dashboards
  * integrações locais

---
📦 **3.5 Banco de Dados Local**

Guarda:
  * usuários
  * permissões
  * auditorias
  * configurações
  * inventário e metadados

---
📦 **3.6 LDAP (opcional)**

Integração com ambiente corporativo do cliente para login centralizado.

---
🪣 **3.7 Repositório S3 (AWS)**

Ponto de distribuição global para os clientes:
  * crl.json (Certificate Revocation List)

⚠️ **Atenção**:
> O ambiente do cliente acessa exclusivamente o S3.<br>
> Não existe comunicação direta do cliente com a intranet da Peta.

---
## 🔄 Fluxo Geral do Licenciamento

1. A Peta gera a licença no ambiente central.
2. A Peta gera e envia a chave pública para o cliente.
3. O cliente faz upload do arquivo .lic na Intellistor Platform.
4. API Licensing Client valida:
   * integridade da assinatura (Ed25519)
   * validade temporal
   * módulos permitidos
   * CRL atualizada
5. Se válido → libera módulos e envia status para o Auth.
6. A Intellistor Platform ativa ou desativa módulos conforme .lic.

---
## 📝 Interpretação Arquitetural (executiva)

* A Peta é autoridade licenciadora.
* O cliente opera totalmente offline, exceto pelo acesso ao S3.
* Licensing Client é o guardião local da validação.
* Auth é o gatekeeper dos usuários.
* Management + Integrator + Backup + Analytics formam os módulos executores.
* O ecossistema é modular e licenciado individualmente.

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com](https://www.intellistor.com)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados

        
