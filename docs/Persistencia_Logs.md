# 📘 Persistência de Logs nas APIs Intellistor - Guia Oficial

A persistência externa de logs é um requisito fundamental para auditoria, troubleshooting e observabilidade corporativa.  
Este guia descreve como configurar corretamente a escrita de logs **fora dos contêineres Docker**, garantindo retenção mesmo após reinicialização ou recriação dos serviços.

Aplicável à todas as APIs:

- 🔑 Auth  
- 🔄 Licensing Client  
- 📡 Integrator  
- 🧭 Management  

---

## 🧩 1. Visão Geral

As APIs da solução escrevem logs em um diretório interno do contêiner (`/opt/app/log`).  
Para garantir persistência e acesso externo, esse diretório deve ser **montado via bind-mount** para um caminho equivalente no host:

```bash
# No host
./logs/<service> 

# No contêiner
/opt/app/log
```

> ⚠️ **Atenção**: Este padrão deve ser aplicado a todos os serviços.

---

## 📂 2. Caminho de Log Interno da Aplicação

O logger da aplicação grava em:

```bash
log/<arquivo>.log
```

E com o `WORKDIR` do contêiner definido como:
```bash
/opt/app
```

O caminho real passa a ser:
```bash
/opt/app/log/<arquivo>.log
```

> ⚠️ **Atenção**: Esse é o diretório que precisa ser mapeado no `docker-compose.yml`.

---

## ⚙️ 3. Configuração do bind-mount no docker-compose

Para cada API, adicione o volume:

Exemplos:
🔄 Licensing Client

```yaml
volumes:
  - ./logs/licensing-client:/opt/app/log
```

🔑 Auth API
```ymal
volumes:
  - ./logs/auth:/opt/app/log
```

📡 Integrator API
```ymal
volumes:
  - ./logs/integrator:/opt/app/log
```

🧭 Management API
```ymal
volumes:
  - ./logs/management:/opt/app/log
```

> 💡 **Dica**: Não precisa criar manualmente as pastas do host — o Docker cria automaticamente.

### 3.1 Exemplo de Configuração completa

```yaml
services:
  # -----------------------------------
  # Licensing Management API (Peta)
  # -----------------------------------
  licensing-management:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: licensing_management
    env_file: .env
    environment:
      SILO_BD: db_mng
      DB_URL: mysql+pymysql://${USER_DB}:${USER_PSS}@db_mng:3306/
      TZ: ${TZ}
      PRIVATE_KEY_PATH: /opt/app/certs/ed25519_private.pem
      PUBLIC_KEY_PATH: /opt/app/certs/ed25519_public.pem
      LIC_DIR: /app/licenses
    expose:
      - "8080"
    restart: unless-stopped
    volumes:
      - ./api_intellistor_licensing/certs:/opt/app/certs
      - ./api_intellistor_licensing/licenses:/app/licenses
      # Persistência 100% automática de logs (sem checklist)
      - ./logs/licensing:/opt/app/log
    depends_on:
      db_mng:
        condition: service_healthy
    networks:
      - peta-net
```

---

## 🚀 4. Aplicando as alterações

Alteração de volumes requer recriação dos contêineres.

Execute:

```bash
docker compose down
docker compose up -d
```

> ⚠️ **Esperado**: Garante que os novos bind-mounts sejam aplicados corretamente.

---
## 🧪 5. Validação da Persistência
### ✔️ 5.1 Verificar se os logs aparecem no host

```bash
ls logs/<service-name>
```

> ⚠️ **Esperado**: Os arquivos devem ser criados quando a API gerar logs.

### ✔️ 5.2 Validar montagem do volume

```bash
docker inspect <container> --format '{{ .Mounts }}'
```

Saída esperada:
```bash
{bind /.../logs/auth /opt/app/log}
```

> ⚠️ **Esperado**: Indica que o bind-mount está ativo.

---

### ✔️ 5.3 Testar escrita em tempo real
```bash
tail -f logs/<service-name>/<arquivo>.log
```

> ⚠️ **Esperado**: Ao acessar qualquer endpoint da API, as entradas devem aparecer instantaneamente.

### ✔️ 5.4 Comparar host × contêiner
Dentro do contêiner:
```bash
docker exec -it <container> ls /opt/app/log
```

No host:
```bash
ls logs/<service-name>
```

> ⚠️ **Esperado**: Ambos devem listar os mesmos arquivos.

---

## 🔁 6. Persistência após recriação

Para confirmar que os logs sobrevivem a reinicializações:

```bash
docker compose down
docker compose up -d
ls logs/<service-name>
```

⚠️ **Esperado**: Os arquivos devem permanecer no host.

---
## 📁 7. Estrutura recomendada de diretórios

A organização oficial dos logs na plataforma:

```text
logs/
   auth/
   licensing-client/
   integrator/
   management/
```

> ⚠️ **Dica**:Cada API mantém seus logs isolados para facilitar observabilidade e auditoria.

---

## 📝 8. Considerações Técnicas

* O caminho **/opt/app/log** deve ser mantido como padrão oficial para todas as APIs.
* Não escreva logs diretamente em caminhos fora desse diretório.
* O uso desse padrão facilita integrações futuras com:
  * 📊 Grafana + Loki
  * 🔍 Pipelines ELK
  * 🛰️ Prometheus exporters
 
---
## ✅ Conclusão

A configuração documentada aqui estabelece o padrão corporativo de persistência de logs para a Intellistor Platform, garantindo:

* Logs persistentes e rastreáveis,
* Diagnóstico avançado,
* Conformidade operacional,
* Consistência entre todos os serviços da solução.

Este é o procedimento oficial a ser seguido por todos os desenvolvedores da plataforma.

---
## 📬 **Contato**

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os mantenedores:

- 🌐 **Site comercial da solução** — [www.intellistor.com.br](https://www.intellistor.com.br)
- 📧 **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- 📧 **Lucas Assis Pereira** — [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- 📧 **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)

© Intellistor Solution – Todos os direitos reservados
