# Seção 5 — Rastreabilidade


## Tabela de Rastreabilidade

| Fatia | Histórias (T2) | Classes principais (Seção 1) | Entidades MER (Seção 2) | Diagrama comportamental (Seção 3) | Casos de teste (Seção 4) |
|---|---|---|---|---|---|
| **Fatia 1** — Registro de atividade com cálculo de calorias | US-SUB2-001, US-SUB2-002, US-SUB2-006 | `Atleta`, `Atividade`, `TipoAtividade`, `CalculadoraCalorias`, `CalculadoraCardio`, `CalculadoraMusculacao`, `Meta` | `ATLETA`, `ATIVIDADE`, `META` | Sequência — `03-comportamental-fatia1.md` | TC-FATIA1-01, TC-FATIA1-02 |
| **Fatia 2** — Ciclo de vida do desafio | US-SUB3-001, US-SUB3-003, US-SUB3-004, US-SUB3-008 | `Atleta`, `AdministradorAcademia`, `Academia`, `Desafio`, `InscricaoDesafio`, `StatusDesafio`, `ServicoNotificacao` | `ATLETA`, `ADMINISTRADOR_ACADEMIA`, `ACADEMIA`, `DESAFIO`, `INSCRICAO_DESAFIO`, `NOTIFICACAO` | Estados — `03-comportamental-fatia2.md` | TC-FATIA2-01, TC-FATIA2-02 |
| **Fatia 3** — Relatório e mensagem de motivação | US-SUB1-004, US-SUB1-005, US-SUB1-007 | `AdministradorAcademia`, `Academia`, `Atleta`, `Atividade`, `Relatorio`, `ItemRelatorio`, `ServicoNotificacao` | `ADMINISTRADOR_ACADEMIA`, `ACADEMIA`, `ACADEMIA_MEMBRO`, `ATIVIDADE`, `RELATORIO`, `ITEM_RELATORIO`, `NOTIFICACAO` | Atividades — `03-comportamental-fatia3.md` | TC-FATIA3-01, TC-FATIA3-02 |

---

