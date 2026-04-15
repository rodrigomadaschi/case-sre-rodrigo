https://github.com/rodrigomadaschi/case-sre-rodrigo


Este repositório contém a solução completa do **case técnico SRE/DevOps da Heineken**, incluindo:

- Pipeline Azure DevOps refatorado  
- Documento de decisão (diagnóstico + soluções)  
- Pipeline equivalente em GitHub Actions (mockado)  
- Estrutura completa para CI/CD seguro, observável e resiliente  

case-sre-rodrigo/
│
├── azure-pipelines.yml              # Pipeline refatorado para Azure DevOps
├── documento-de-decisao.md          # Diagnóstico, decisões e rollback
├── README.md                        # Este arquivo
└── .github/
└── workflows/
└── lagunitas-ci.yml         # GitHub Actions mockado


---

# 🚀 Objetivo do Case

O objetivo é demonstrar como tornar o processo de CI/CD da plataforma de dados da Lagunitas:

- Mais **seguro**  
- Mais **observável**  
- Mais **confiável**  
- Com **rollback rápido**  
- Com **validações automáticas** pós‑deploy  

O foco é evitar incidentes como o descrito no case, onde um deploy incorreto passou despercebido e impactou dados críticos de vendas.

---

# 🔧 Tecnologias Utilizadas

- **Azure DevOps Pipelines**
- **Azure Databricks**
- **Azure Data Factory (ADF)**
- **Azure Log Analytics**
- **GitHub Actions (mock)**
- **Shell Script / Bash**
- **ARM Templates**

---

# 🧪 GitHub Actions – Pipeline Mockado

O workflow `lagunitas-ci.yml` simula:

- Build e lint dos notebooks  
- Deploy mockado de Databricks  
- Deploy mockado de ADF  
- Smoke tests simulados  
- Log estruturado no final  

Nenhum recurso real do Azure é necessário.

### ▶️ Como rodar

Basta fazer um push para a branch `main`:

```bash
git add .
git commit -m "run pipeline"
git push origin main


Principais Melhorias Implementadas
✔ Gate de aprovação antes de PRD
✔ Smoke tests automáticos (Databricks + ADF)
✔ Logs estruturados enviados ao Log Analytics
✔ Estratégia de rollback baseada em SHA
✔ Segregação de credenciais por ambiente
✔ Pipeline modular e seguro

Rollback
A estratégia de rollback cobre:

Databricks: redeploy da versão anterior via commit SHA

ADF: reaplicação do ARM template anterior

Smoke tests pós‑rollback

RTO ≤ 15 minutos

Detalhes completos no arquivo:
documento-de-decisao.md

Documentação Completa
O documento de decisão inclui:

Diagnóstico do incidente

Causas técnicas e de processo

Pipeline refatorado

Observabilidade

Rollback

Considerações finais


Contato
Rodrigo  
Candidato ao processo SRE/DevOps – Heineken
Campinas, SP – Brasil