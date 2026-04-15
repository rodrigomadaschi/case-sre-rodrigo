# Documento de Decisão – Case Técnico SRE/DevOps  
**Empresa:** Heineken / Lagunitas  
**Autor:** Rodrigo  
**Data:** 14/04/2026  

---

# 1. Introdução

Este documento apresenta o diagnóstico do incidente ocorrido na plataforma de dados da Lagunitas, bem como a proposta de um pipeline CI/CD refatorado, com foco em confiabilidade, governança e observabilidade.  
O objetivo é garantir que falhas de deploy sejam detectadas automaticamente, evitando impacto ao negócio e reduzindo o tempo de recuperação.

---

# 2. Diagnóstico do Incidente

## 2.1 Causas Técnicas

| Problema | Impacto |
|---------|---------|
| Deploy sem validação do workspace | Notebooks foram publicados em DEV em vez de PRD |
| Token com permissão em múltiplos ambientes | Pipeline conseguiu escrever no workspace errado |
| Ausência de smoke tests | Deploy incorreto não foi detectado |
| Deploy ADF sem validação pós‑deploy | Pipelines ficaram inativos sem alerta |
| Falta de logs estruturados | RCA demorou 40 minutos |
| Sem rollback automatizado | Remediação manual levou 42 minutos |

---

## 2.2 Causas de Processo

| Problema | Impacto |
|---------|---------|
| Merge na main = deploy direto em PRD | Falha chegou à produção sem revisão |
| ACC não usado como gate | Ambiente intermediário não cumpriu papel |
| Falta de segregação de credenciais | Token único permitiu erro crítico |
| Cultura de observabilidade insuficiente | Pipeline “verde” era tratado como sucesso |
| Dependência de stakeholders | Falha descoberta pelo time de negócios |

---

## 2.3 Por que o pipeline ficou verde

O pipeline reportou sucesso porque **nenhum step verificava o estado real do deploy**.  
Todos os comandos retornaram exit code 0, mesmo com o deploy incorreto.

O pipeline deveria ter validado:

### Databricks
- Workspace = PRD  
- Notebooks no path correto  
- Job ativo e apontando para a versão recém‑deployada  

### ADF
- Pipelines habilitados  
- Triggers ativas  
- Deploy aplicado no factory correto  

---

## 2.4 Limitações do lint atual

### O que ele detecta
- Erros de sintaxe Python  
- Arquivos corrompidos  
- Indentação inválida  

### O que ele NÃO detecta
- Erros de lógica  
- Falhas de integração  
- Dependências ausentes  
- Erros em `%sql`, `%scala`, `%md`  
- Problemas de runtime  
- Paths incorretos  
- Configuração incorreta de jobs  

---

## 2.5 A melhoria mais importante

**Adicionar smoke tests pós‑deploy.**

Motivo:  
Foi exatamente a ausência deles que permitiu que o deploy errado chegasse a PRD.  
Mesmo com gate, logs e rollback, o problema teria ocorrido sem validação automática.

---

# 3. Pipeline Refatorado

O pipeline refatorado (anexo ao final) inclui:

- Gate de aprovação antes de PRD  
- Smoke tests Databricks + ADF  
- Logs estruturados enviados ao Log Analytics  
- Estrutura modular e clara  
- Deploy seguro e rastreável  

---

## 3.1 Melhorias Implementadas

| Área | Melhoria |
|------|----------|
| Governança | Gate de aprovação com environment PRD |
| Segurança | Segregação de credenciais por ambiente |
| Observabilidade | Log estruturado com commit, autor, ambiente e smoke tests |
| Confiabilidade | Smoke tests automáticos pós‑deploy |
| Operação | Estratégia de rollback baseada em SHA |

---

## 3.2 Gate de Aprovação

O environment `prd-approval-env` deve ter:

- Tech Lead  
- SRE responsável  
- PO da plataforma  

O aprovador deve visualizar:

- Commit SHA  
- Diff da PR  
- Resultado do Build  
- Artefatos gerados  
- Histórico do pipeline  

---

## 3.3 Smoke Tests

### Databricks
- Workspace correto  
- Notebooks no path esperado  
- Job ativo e apontando para a versão correta  

### ADF
- Pipelines habilitados  
- Triggers ativas  
- Deploy aplicado no factory correto  

Nenhum smoke test executa dados reais.

---

## 3.4 Observabilidade

O pipeline envia um log estruturado contendo:

- timestamp  
- commit SHA  
- ambiente  
- resultado dos smoke tests  
- responsável pelo merge  

### Destino recomendado: **Azure Log Analytics**

Justificativas:

- Permite consultas com KQL  
- Integra com Azure Monitor  
- Suporta dashboards e alertas  
- Armazena histórico de deploys  

---

# 4. Estratégia de Rollback

## 4.1 Databricks

### O que é possível automatizar
- Redeploy da versão anterior usando o commit SHA  
- Reimportação dos notebooks via `import_dir`  
- Reexecução dos smoke tests  

### Limitações sem Asset Bundles
- Sem versionamento nativo  
- Sem atomicidade  
- Sem preview de alterações  

---

## 4.2 Azure Data Factory

### O que o Azure NÃO guarda
- Histórico de ARM templates  
- Versões anteriores de pipelines  

### O que precisa ser armazenado
- ARMTemplateForFactory.json  
- ARMTemplateParametersForFactory.json  
- Commit SHA  

### Fluxo de rollback
1. Selecionar SHA anterior  
2. Reaplicar ARM template  
3. Validar pipelines e triggers  
4. Registrar log estruturado  

---

## 4.3 RTO – Recovery Time Objective

### Recomendação: **≤ 15 minutos**

| Componente | Tempo |
|-----------|--------|
| Databricks rollback | 3–5 min |
| ADF rollback | 5–7 min |
| Smoke tests | 2–3 min |
| **Total** | **10–15 min** |

A estratégia atende o RTO desejado.

---

# 5. Conclusão

O pipeline original falhou por ausência de validações, governança e observabilidade.  
O pipeline refatorado:

- Evita deploys incorretos  
- Detecta falhas automaticamente  
- Gera logs estruturados  
- Permite rollback rápido  
- Aumenta a confiabilidade da plataforma  

A operação torna‑se previsível, auditável e resiliente — exatamente o que se espera de um SRE.

---

# 6. Anexos

## 6.1 azure-pipelines.yml  
*(arquivo incluído no repositório)*

## 6.2 GitHub Actions (mock)  
*(arquivo incluído no repositório)*

