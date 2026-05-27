# Seção 0 — Seleção de Escopo

## 0.1 Subsistemas e atores identificados (recapitulação dos Trabalhos 1 e 2)

O FitTrack está organizado em três subsistemas principais:

| Subsistema | Atores principais |
|---|---|
| SUB1 — Gestão de Academia | Administrador da Academia |
| SUB2 — Rastreamento de Atividades | Atleta |
| SUB3 — Desafios e Comunidade | Administrador da Academia, Atleta |

---

## 0.2 Fatias selecionadas

### Fatia 1 — Atleta registra atividade física com cálculo de calorias

**Histórias cobertas:** US-SUB2-001 ("Como atleta, quero registrar uma atividade física informando tipo, duração e distância"), US-SUB2-002 ("Como atleta, quero visualizar a estimativa de calorias queimadas após registrar uma atividade"), US-SUB2-006 ("Como atleta, quero informar meus dados físicos (peso e altura) para que o sistema calcule com maior precisão as calorias queimadas").

**Por que é representativa:**
- Must Have no MoSCoW: sem o registro de atividade, nenhum outro fluxo do sistema existe.
- Atravessa dois subsistemas: SUB2 (registro e cálculo) e indiretamente o perfil do atleta em SUB1 (dados físicos persistidos).
- Contém regra de negócio não-trivial: o cálculo de calorias depende do tipo de atividade, da duração e dos dados físicos do usuário (peso e altura), e o resultado deve ser apresentado como estimativa, o que impõe decisões de design sobre fórmulas e validação de entrada.
- Possui caminhos de exceção: dados físicos ausentes, distância não informada para musculação, valores fora de limites plausíveis.

**O que esperamos aprender:**
- Como representar um fluxo de registro com validação condicional por tipo de atividade.
- Como modelar a dependência entre dados de perfil (dados físicos) e cálculo de uma operação posterior (registro de atividade).
- Como expressar alternativas (`alt`) e guardas em diagrama de sequência para o cálculo condicional de calorias.

---

### Fatia 2 — Ciclo de vida de um desafio: da criação à conclusão

**Histórias cobertas:** US-SUB3-001 ("Como administrador, quero criar desafios para os membros da minha academia"), US-SUB3-003 ("Como atleta, quero me inscrever em um desafio para que minhas atividades sejam contabilizadas automaticamente"), US-SUB3-004 ("Como atleta, quero acompanhar meu progresso e o ranking em tempo real"), US-SUB3-008 ("Como administrador, quero encerrar um desafio antes do prazo").

**Por que é representativa:**
- Envolve dois atores distintos (Administrador e Atleta) e dois subsistemas (SUB1 e SUB3), exercitando interações entre perfis e módulos.
- Apresenta um ciclo de vida bem definido com transições de estado: Rascunho → Ativo → Em andamento → Encerrado → Arquivado. Cada transição tem evento, regra e ação automática (notificações, bloqueio de contabilização).
- Contém regras de negócio não-triviais: o atleta não pode se inscrever duas vezes; atividades registradas antes da inscrição não são contabilizadas; após encerramento nenhuma nova atividade é aceita; o administrador pode encerrar antecipadamente.
- É Must Have no MoSCoW (US-SUB3-001, US-SUB3-003, US-SUB3-004).

**O que esperamos aprender:**
- Como modelar um ciclo de vida com vários estados intermediários e transições com guardas.
- Como diferenciar "estado do desafio" de "estado da inscrição do atleta", são entidades diferentes com ciclos de vida próprios.
- A diferença prática entre usar diagrama de estados (para o ciclo de vida do desafio) versus diagrama de atividades (para o fluxo de inscrição).

---

### Fatia 3 — Administrador visualiza relatório de atividades dos alunos

**Histórias cobertas:** US-SUB1-004 ("Como administrador, quero visualizar o relatório de atividades dos meus alunos para identificar quem está engajado e quem precisa de incentivo"), US-SUB1-007 ("Como administrador, quero visualizar um ranking de engajamento dos meus alunos"), US-SUB1-005 ("Como administrador, quero enviar mensagens de motivação para os meus alunos").

**Por que é representativa:**
- Envolve múltiplos subsistemas: SUB1 (geração do relatório e envio de mensagens) consome dados do SUB2 (histórico de atividades dos atletas), exercitando a fronteira entre módulos.
- É a principal entrega de valor para o Administrador: o caso justifica o uso da plataforma pela academia e é Must Have no MoSCoW.
- Contém regras de negócio não-triviais: o administrador acessa apenas dados de sua própria academia (isolamento por tenant); o relatório agrega dados de múltiplos atletas; o ranking é calculado dinamicamente sobre atividades no período selecionado; mensagens de motivação só podem ser enviadas aos membros ativos da academia.
- Expõe uma dimensão diferente das duas primeiras fatias: enquanto a Fatia 1 é centrada em um único ator, e a Fatia 2 tem ciclo de vida complexo, a Fatia 3 é um fluxo distribuído com agregação de dados e controle de escopo por tenant.

**O que esperamos aprender:**
- Como modelar um fluxo com swimlanes (Administrador, Sistema, SUB2 - fonte de dados).
- Como representar decisões compostas: filtro por período, por tipo de atividade, por academia.
- Como evidenciar a fronteira entre subsistemas numa consulta agregada.

---

## 0.3 Cobertura dos critérios obrigatórios

| Critério | Fatia 1 | Fatia 2 | Fatia 3 |
|---|---|---|---|
| Must Have do MoSCoW | ✅ Sim | ✅ Sim | ✅ Sim |
| Múltiplos subsistemas / atores | SUB1 + SUB2 | SUB1 + SUB3, 2 atores | SUB1 + SUB2, 2 atores |
| Regras de negócio não-triviais | Cálculo condicional de calorias | Estados + transições com guardas | Isolamento por tenant + agregação |

---

## 0.4 O que fica de fora (e por quê)

- **Cadastro e autenticação** (US-SUB1-001, US-SUB1-002): fluxo padrão de cadastro com envio de e-mail de convite. Não há regra de negócio específica do domínio de fitness; modelar não traria insight novo além do que já é capturado na Fatia 3.
- **Importação de wearables** (US-SUB2-008): classificada como Won't Have no MoSCoW do MVP. Depende de APIs proprietárias (Google Fit, Apple Health) e seria uma caixa-preta de integração externa; não exercita design de domínio.
- **Visualização de trajeto no mapa** (US-SUB2-009): Could Have no MoSCoW. Requer integração com serviço de mapas e GPS; a modelagem seria dominada pela integração externa, não pelo domínio.
- **Compartilhamento de resultado** (US-SUB3-005):  escopo por tenant escopo por tenant. Funcionalidade de geração de card e compartilhamento social; operação simples e isolada sem regra de negócio relevante para o domínio.
- **Painel administrativo da plataforma** (US-SUB1-006, US-SUB1-008, US-SUB1-009, US-SUB1-010): operações predominantemente CRUD de configuração. Relatórios e ranking já são cobertos pela Fatia 3.

---

## 0.5 Histórias modeladas neste trabalho

| Fatia | Histórias de usuário cobertas |
|---|---|
| Fatia 1 | US-SUB2-001, US-SUB2-002, US-SUB2-006 |
| Fatia 2 | US-SUB3-001, US-SUB3-003, US-SUB3-004, US-SUB3-008 |
| Fatia 3 | US-SUB1-004, US-SUB1-005, US-SUB1-007 |
