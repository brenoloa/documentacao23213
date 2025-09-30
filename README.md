# VivaCond — Documentação de Projeto
_Data: 12/09/2025_

> **Escopo deste documento**: visão geral do projeto, riscos e plano de gerenciamento.
---

## 1) Descrição do Projeto

**Nome:** VivaCond — Sistema de Gerenciamento de Condomínios  
**Objetivo:** Digitalizar e facilitar rotinas de condomínios (administração, comunicação com moradores, reservas de áreas, ocorrências, avisos, pagamentos/integrações financeiras, etc.) com foco em usabilidade, segurança e rastreabilidade.

**Arquitetura (alta visão):**
- **Frontend:** Angular (SPA), estilização com Tailwind/DaisyUI, máscara de campos (ngx-mask), build para hospedagem estática (ex.: Vercel/S3/CloudFront).
- **Backend:** Spring Boot 3.x (Java 17+), API REST com autenticação JWT, CORS configurado por ambiente, JPA/Hibernate.
- **Banco de Dados:** PostgreSQL (produção), H2 para desenvolvimento local (opcional).
- **CI/CD:** pipeline com build, testes, análise estática e deploy automatizado por branch/tag.
- **Observabilidade:** logs estruturados; métricas/health-checks; monitoramento de erros (ex.: Sentry) — a configurar.
- **Segurança:** JWT; perfis de acesso (síndico, administrador, porteiro, morador); política de CORS e variáveis de ambiente; aderência à LGPD.

**Módulos/funcionalidades (exemplos):**
- Cadastros (condomínio, blocos, unidades, moradores, funcionários).
- Comunicação (murais, avisos, notificações, chat interno básico).
- Reservas de áreas comuns.
- Ocorrências/solicitações e acompanhamento.
- Financeiro básico / integrações (a depender do escopo fechado).
- Relatórios e auditoria (histórico de ações).

---

## 2) Metodologia Ágil para Desenvolvimento

**Abordagem:** _Scrumban_ (Scrum + Kanban) — mantemos sprints curtas e cerimônias do Scrum, com um quadro Kanban para fluxo contínuo e limites de WIP.

### 2.1 Papéis
- **Product Owner (PO):** prioriza backlog, define valor de negócio e critérios de aceite.
- **Scrum Master (SM):** remove impedimentos, promove melhoria contínua.
- **Time de Desenvolvimento:** frontend, backend, QA, DevOps; time multifuncional.
- **Stakeholders:** síndicos, administradoras, porteiros, moradores, diretoria.

### 2.2 Cerimônias (cadência quinzenal sugerida)
- **Planejamento da Sprint (2h):** selecionar itens prontos (DoR) e estimar com **story points**.
- **Daily (15m):** status, impedimentos, plano diário.
- **Refinamento (1h/semana):** detalhar histórias, anexar critérios de aceite, quebrar tarefas.
- **Review (1h):** demonstrar incrementos aos stakeholders.
- **Retrospectiva (1h):** o que manter, mudar, iniciar, parar.

### 2.3 Artefatos e Definições
- **Product Backlog:** épicos, features e histórias priorizadas por valor/risco.
- **Sprint Backlog:** metas da sprint e tarefas.
- **Definition of Ready (DoR):** história bem descrita, dependências mapeadas, mock/aceite definidos.
- **Definition of Done (DoD):** código com testes, review aprovado, cobertura mínima, build verde, documentação/CHANGELOG atualizados, deploy em ambiente de homologação.

### 2.4 Qualidade e Automação
- **Pirâmide de testes:** unitários (JUnit/Jest), integração (Spring + Testcontainers), E2E (Cypress/Playwright).
- **CI/CD:** gatilhos por PR/main; *quality gates* (linters, cobertura, SAST/DAST quando possível).
- **Revisão de código:** PR com 1+ aprovadores; _branching_ leve (trunk-based ou GitFlow simplificado).

### 2.5 Gestão de Requisitos e UX
- **Discovery rápido:** protótipos de baixa fidelidade para validar fluxos críticos (reservas, ocorrências).
- **Critérios de aceite orientados a comportamento (Gherkin/BDD)** quando fizer sentido.
- **Acessibilidade:** contrastes, navegação por teclado, rótulos ARIA.

---

## 3) Gestão de Riscos do Projeto

### 3.1 Escala de Probabilidade e Impacto
- **Probabilidade (P):** 1=Muito baixa … 5=Muito alta
- **Impacto (I):** 1=Baixo (quase sem efeito) … 5=Crítico (interrompe operações)
- **Exposição (E):** `E = P × I` (priorizar E mais alto)

### 3.2 Registro de Riscos
| ID | Risco | Categoria | Causa | Consequência | P | I | E | Disparadores | Mitigações | Contingência | Responsável | Status |
|----|-------|-----------|-------|--------------|---|---|---|--------------|------------|--------------|-------------|--------|
| R1 | Vazamento de dados (LGPD) | Segurança | Configuração incorreta de CORS/credenciais | Multas/reputação/indisponibilidade | 3 | 5 | 15 | alertas de WAF, picos de tráfego | _Secrets_ via **ENV/Secrets Manager**, _least privilege_, auditoria de acesso, pentest | isolar serviço, rotacionar chaves, notificar DPO e usuários | Eng. Líder | Aberto |
| R2 | Instabilidade do banco | Infra | picos de uso ou índices ausentes | lentidão/queda | 3 | 4 | 12 | aumento de latência | auto‐scaling/planos maiores, índices e _query plan_ | failover, réplicas de leitura | DevOps/DBA | Aberto |
| R3 | Atrasos por escopo difuso | Escopo | requisitos mudam sem controle | atrasos/custos | 4 | 3 | 12 | histórias grandes/ambíguas | DoR rígida, _change control_, épicos fatiados | replanejar sprint e MVP | PO/SM | Aberto |
| R4 | Baixa adoção pelos usuários | Produto/UX | fluxos complexos | retrabalho/treinamento | 3 | 3 | 9 | tickets repetitivos | testes de usabilidade, _walkthroughs_ | guias em vídeo, _feature flags_ | UX/PO | Aberto |
| R5 | Dependência de terceiros | Integração | APIs externas instáveis | interrupções | 2 | 4 | 8 | erros 5xx/timeout | _circuit breaker_, _retry/backoff_, _mock_ | fila de compensação | Backend | Aberto |
| R6 | Falhas de backup/restore | Operação | política mal definida | perda de dados | 2 | 5 | 10 | jobs com erro | _backups_ diários + teste de **restore** | restore documentado | DevOps | Aberto |
| R7 | Rotatividade no time | Pessoas | saída de membros-chave | perda de conhecimento | 3 | 3 | 9 | queda na cadência | _pairing_, _docs_, _bus factor_ | _onboarding_ rápido | SM | Aberto |
| R8 | Bugs críticos em produção | Qualidade | baixa cobertura de testes | incidentes | 3 | 4 | 12 | falhas repetidas | testes/E2E, _feature toggles_, canário | rollback rápido | Eng. Líder | Aberto |

> **Atenção:** Nunca versionar _secrets_. Armazenar credenciais exclusivamente em variáveis de ambiente/secret manager. Parâmetros sensíveis devem ser **mascarados** em logs e fora do repositório.

### 3.3 Plano de Resposta e Monitoramento
- **Revisão quinzenal** do registro de riscos na Retrospectiva/Planejamento.
- **Métricas de risco:** bugs críticos por sprint, _lead time_ de correções, _MTTR_, cobertura de testes.
- **Indicadores de segurança:** OWASP Top 10, varreduras SAST/DAST, cabeçalhos HTTP seguros, _rate limit_.

---

## 4) Plano de Entrega (MVP → Incrementos)

**MVP (exemplo):**
- Autenticação JWT (papéis básicos), cadastro de condomínio/unidade/morador, reservas simples, mural de comunicados, ocorrências básicas, auditoria mínima.

**Incrementos sugeridos:**
1. **Financeiro básico** (integração/faturamento, exportações).
2. **Notificações** (email/push).
3. **Relatórios e BI leve**.
4. **Portaria** (controle de visitantes/com entregas).
5. **Votações/assembleias** (quando aplicável).

**Ambientes:**
- **Dev:** banco efêmero (H2/postgres local), _feature flags_ liberais.
- **Homologação:** dados de teste, _smoke tests_, aprovações de PO.
- **Produção:** _blue/green_ ou canário, observabilidade e _alerts_.

---

## 5) Padrões Técnicos e Operacionais

- **Configuração por ambiente** (ENV/Profiles): `API_URL`, `JWT_SECRET`, `DB_URL`, `CORS_ORIGINS`…
- **Logs**: estruturados (JSON), correlação por `traceId`.
- **Segurança**: CORS por domínio; cabeçalhos (CSP, HSTS, X-Content-Type-Options, etc.); _rate limiting_.
- **Banco**: versionamento de schema (Flyway/Liquibase), índices e migrações.
- **Front**: lazy-loading, _guards_, interceptors (Auth/HTTP), _skeletons_ e acessibilidade.
- **Back**: DTOs/records, validações (`@Valid`), _error handler_ global, paginação, idempotência onde aplicável.
- **Documentação de API**: OpenAPI/Swagger com exemplos e _schemas_.

---
