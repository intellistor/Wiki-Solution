# 📦 Processo de Deploy para Produção

Este documento descreve o processo de liberação para produção dos projetos da equipe, garantindo qualidade, rastreabilidade e segurança nas entregas.

---

## ✅ Critérios de Pronto para Produção

Antes de iniciar o deploy, certifique-se de que:

- [ ] Passou por revisão de código (code review)
- [ ] Testes automatizados passaram com sucesso (unitário, integração, funcional)
- [ ] Cobertura de linhas >= 90%
- [ ] Funcionalidade validada pelo PO em ambiente de homologação
- [ ] Changelog atualizado com as alterações da versão
- [ ] Versão definida (ex: `v1.2.0`)
- [ ] Backup realizado (se aplicável)

**Importante**: Use uma checklist de _“Definition of Done”_ para garantir consistência.

---

## 🛠️ Ambientes

- **Desenvolvimento/Local (Dev)**: Testes técnicos e validações iniciais
- **Homologação (Staging)**: Validação funcional e aceitação do PO
- **Produção (Prod)**: Ambiente final, acessado por usuários reais

---

## 🔁 Pipeline de Deploy

Fluxo automatizado via CI/CD:

1. Commit no repositório principal
2. Execução de testes automatizados
3. Build e versionamento
4. Deploy automático em Staging
5. Validação final
6. Deploy manual ou automatizado em Produção

Ferramentas recomendadas: GitHub Actions e/ou AWS (_em implementação_)

---

## 📅 Janela de Liberação

- Dia padrão: Quintas-feiras
- Horário: Entre 16h e 18h
- Gatilho: Fim da Sprint e aprovação do PO

---

## 🧪 Pós-Deploy

Após a liberação:

- [ ] Monitorar logs e métricas
- [ ] Validar funcionalidades críticas
- [ ] Comunicar _stakeholders_ sobre a nova versão
- [ ] Registrar versão e data no histórico de deploys

---

## 🔙 Rollback

Em caso de falha:

- Reverter para versão anterior
- Desativar feature via flag (se disponível)
- Comunicar equipe e registrar ISSUE (detalhar falha)

---
##
Contato

Em caso de dúvidas, sugestões ou contribuições, entre em contato com os responsáveis pelo projeto:

- **Eloi Salton** — [eloi.externo@petacorp.com.br](mailto:eloi.externo@petacorp.com.br)
- **Lucas Assis Pereira** - [lucas.pereira@petacorp.com.br](lucas.pereira@petacorp.com.br)
- **Renato de Carvalho Machado** — [renato.externo@petacorp.com.br](mailto:renato.externo@petacorp.com.br)
---

