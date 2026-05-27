# Seção 4 — Casos de Teste

Para cada uma das três fatias foram elaborados dois casos de teste seguindo o padrão IEEE-830, totalizando seis casos. Em cada par, pelo menos um caso cobre um **cenário de fronteira ou caminho de erro** — não apenas o caminho feliz.

---

## Fatia 1 — Registro de atividade com cálculo de calorias

### TC-FATIA1-01 — Registro com cálculo de calorias ("caminho feliz")

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA1-01 |
| **Fatia / História** | Fatia 1 — US-SUB2-001 (registrar atividade) e US-SUB2-002 (visualizar estimativa de calorias) |
| **Pré-condições** | Atleta autenticado; dados físicos cadastrados: peso = 75 kg, altura = 175 cm; nenhuma meta ativa do tipo distância. |
| **Dados de entrada** | Tipo: CORRIDA; Duração: 45 minutos; Distância: 6,5 km. |
| **Passos** | 1) Atleta acessa tela "Registrar Atividade". 2) Seleciona tipo CORRIDA. 3) Informa duração 45 min e distância 6,5 km. 4) Clica em "Salvar". |
| **Resultado esperado** | Sistema registra a atividade; exibe confirmação com estimativa de calorias (valor numérico calculado com base no peso do atleta e na duração); exibe nota "\* valor estimado"; atividade aparece no histórico imediatamente. |
| **Critério de aprovação** | (a) HTTP 201 retornado com `caloriasEstimadas > 0`. (b) Atividade visível no histórico com todos os campos preenchidos. (c) Mensagem de estimativa presente na interface. (d) Dados físicos do atleta não foram modificados. |
| **Severidade em caso de falha** | Alta — falha aqui significa que a funcionalidade nuclear do produto não opera corretamente. |

---

### TC-FATIA1-02 — Registro de corrida sem distância informada (fronteira de validação)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA1-02 |
| **Fatia / História** | Fatia 1 — US-SUB2-001 (registrar atividade) |
| **Pré-condições** | Atleta autenticado; dados físicos cadastrados. |
| **Dados de entrada** | Tipo: CORRIDA; Duração: 30 minutos; Distância: *campo vazio / não informado*. |
| **Passos** | 1) Atleta acessa tela "Registrar Atividade". 2) Seleciona tipo CORRIDA. 3) Informa duração 30 min. 4) Deixa campo distância em branco. 5) Clica em "Salvar". |
| **Resultado esperado** | Sistema rejeita o registro com mensagem de erro clara: "Distância é obrigatória para atividades de cardio". Nenhuma atividade é persistida. Formulário permanece com os dados preenchidos para correção. |
| **Critério de aprovação** | (a) HTTP 400 retornado com campo `errors` descrevendo a validação falha. (b) Nenhum registro inserido na tabela `ATIVIDADE` para o atleta nesta requisição. (c) Mensagem de erro exibida no campo distância, não apenas em toast genérico. |
| **Severidade em caso de falha** | Média — falha aqui significa persistência de dados incompletos que corrompem histórico e estatísticas do atleta. |

---

## Fatia 2 — Ciclo de vida de um desafio

### TC-FATIA2-01 — Atleta se inscreve e tem progresso contabilizado ("caminho feliz")

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA2-01 |
| **Fatia / História** | Fatia 2 — US-SUB3-003 (inscrever-se em desafio) e US-SUB3-004 (acompanhar progresso e ranking) |
| **Pré-condições** | Academia "Alpha Fitness" cadastrada; Desafio "Corrida de Maio" com status ATIVO, métrica DISTANCIA, meta 50 km, prazo = último dia do mês; Atleta "Maria" autenticada e membro da academia; nenhuma inscrição prévia de Maria neste desafio. |
| **Dados de entrada** | Atleta: Maria; Desafio: "Corrida de Maio"; Atividade pós-inscrição: CORRIDA, 8 km. |
| **Passos** | 1) Maria acessa a listagem de desafios. 2) Seleciona "Corrida de Maio". 3) Clica em "Inscrever-se". 4) Sistema confirma inscrição. 5) Maria registra atividade CORRIDA, 8 km. 6) Maria acessa o ranking do desafio. |
| **Resultado esperado** | Inscrição confirmada com mensagem de boas-vindas. Após registro da atividade, o progresso de Maria no desafio mostra 8,0 km de 50 km. Ranking exibe Maria com posição calculada em relação aos demais participantes. |
| **Critério de aprovação** | (a) `InscricaoDesafio` criada com `progressoAcumulado = 0.0` no momento da inscrição. (b) Após registro da atividade, `progressoAcumulado = 8.0`. (c) Ranking retorna posição de Maria. (d) Atividade registrada antes da inscrição não é contabilizada retroativamente. |
| **Severidade em caso de falha** | Crítica — falha aqui significa que o subsistema de desafios não opera; é Must Have do MoSCoW. |

---

### TC-FATIA2-02 — Atleta tenta inscrição duplicada no mesmo desafio (transição crítica de estado)

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA2-02 |
| **Fatia / História** | Fatia 2 — US-SUB3-003 (inscrição com guarda "atleta não inscrito anteriormente") |
| **Pré-condições** | Desafio "Corrida de Maio" com status ATIVO; Atleta "Maria" já inscrita com `InscricaoDesafio` confirmada. |
| **Dados de entrada** | Atleta: Maria; segunda tentativa de inscrição no desafio "Corrida de Maio". |
| **Passos** | 1) Maria acessa a tela do desafio "Corrida de Maio" (já inscrita). 2) Tenta clicar novamente em "Inscrever-se" (ex.: via chamada direta à API, contornando o botão já oculto na UI). |
| **Resultado esperado** | Sistema rejeita a segunda inscrição com mensagem clara. Nenhuma nova linha em `INSCRICAO_DESAFIO` é criada. O progresso e a posição da inscrição existente permanecem inalterados. |
| **Critério de aprovação** | (a) HTTP 409 Conflict retornado com mensagem "Atleta já inscrito neste desafio". (b) Apenas uma linha em `INSCRICAO_DESAFIO` para o par (Maria, Corrida de Maio) — validado via consulta ao banco. (c) Progresso acumulado original não alterado. |
| **Severidade em caso de falha** | Alta — inscrições duplicadas corrompem o ranking e a contabilização de progresso. |

---

## Fatia 3 — Relatório de atividades e mensagem de motivação

### TC-FATIA3-01 — Geração de relatório mensal com dados ("caminho feliz")

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA3-01 |
| **Fatia / História** | Fatia 3 — US-SUB1-004 (visualizar relatório de atividades) e US-SUB1-007 (ranking de engajamento) |
| **Pré-condições** | Academia "Alpha Fitness" cadastrada; Administrador "Carlos" autenticado como gestor dessa academia; 3 atletas membros: Ana (5 atividades no mês), Bruno (3 atividades no mês), Carla (8 atividades no mês); período selecionado: maio/2026. |
| **Dados de entrada** | Período: MENSAL (maio/2026); Tipos de atividade: todos. |
| **Passos** | 1) Carlos acessa painel de relatórios. 2) Seleciona período Mensal — maio/2026. 3) Mantém todos os tipos selecionados. 4) Clica em "Gerar Relatório". |
| **Resultado esperado** | Relatório gerado e exibido com: Carla (1º, 8 atividades), Ana (2º, 5 atividades), Bruno (3º, 3 atividades). Totais de distância e calorias estimadas por atleta exibidos. Relatório persistido para exportação futura. |
| **Critério de aprovação** | (a) Ranking correto por total de atividades no período. (b) Somente atletas da academia "Alpha Fitness" aparecem no relatório — nenhum atleta de outra academia. (c) Uma linha em `RELATORIO` e três linhas em `ITEM_RELATORIO` persistidas no banco. (d) Tempo de resposta inferior a 5 segundos (NF-SUB2-001). |
| **Severidade em caso de falha** | Alta — falha aqui significa que a principal entrega de valor para o administrador não funciona. |

---

### TC-FATIA3-02 — Administrador tenta acessar relatório de outra academia ("valor limítrofe de autorização")

| Campo | Conteúdo |
|---|---|
| **ID** | TC-FATIA3-02 |
| **Fatia / História** | Fatia 3 — US-SUB1-004 + NF-SUB1-004 (controle de acesso e isolamento de dados por academia) |
| **Pré-condições** | Duas academias cadastradas: "Alpha Fitness" (id=1) e "Beta Sports" (id=2). Administrador "Carlos" autenticado como gestor da academia id=1. Relatório existente gerado para academia id=2. |
| **Dados de entrada** | Requisição: `GET /relatorios?academia_id=2&periodo=MENSAL` com token de Carlos (academia_id=1 no token). |
| **Passos** | 1) Carlos, autenticado, constrói manualmente (ex.: via Postman ou DevTools) uma requisição solicitando relatório da academia id=2. 2) Envia a requisição ao backend. |
| **Resultado esperado** | Sistema rejeita a requisição com HTTP 403 Forbidden. Nenhum dado da academia "Beta Sports" é retornado. Nenhuma informação que confirme ou negue a existência da academia id=2 é exposta. |
| **Critério de aprovação** | (a) HTTP 403 retornado sem dados de conteúdo sensível no body. (b) Log de auditoria registra tentativa de acesso não autorizado com timestamp, userId e recurso solicitado. (c) Dados da academia "Beta Sports" permanecem inacessíveis em qualquer endpoint consultado por Carlos. |
| **Severidade em caso de falha** | Crítica — falha aqui é uma brecha de segurança e viola diretamente a LGPD (isolamento de dados sensíveis entre tenants). |

---
