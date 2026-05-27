# Seção 3 — Modelagem Comportamental: Fatia 3

## Fatia 3 — Administrador visualiza relatório e envia mensagem de motivação

**Opção C — Diagrama de Atividades**

Justificativa da escolha: A Fatia 3 envolve um fluxo de trabalho com múltiplos atores (Administrador e Sistema) e múltiplas decisões em sequência: selecionar período, selecionar tipos de atividade, gerar dados agregados, visualizar ranking e opcionalmente enviar mensagem. O diagrama de atividades é o mais adequado porque: (a) o fluxo tem ramificações condicionais (ex.: nenhuma atividade no período → exibir estado vazio) e paralelismo (geração de relatório e envio de mensagem são independentes); (b) as swimlanes evidenciam claramente a fronteira entre o que é responsabilidade do Administrador e o que é processado pelo Sistema — especialmente a fronteira entre SUB1 (geração de relatório) e SUB2 (fonte dos dados de atividade); (c) não há ciclo de vida de entidade com estados discretos que justificasse um diagrama de estados, e não há coordenação síncrona entre serviços distintos que favorecesse um diagrama de sequência.

---

## Diagrama de Atividades

```mermaid
flowchart TD
    subgraph Admin["🧑‍💼 Administrador da Academia"]
        A([Início: acessa painel de relatórios]) --> B[Seleciona período\nSemanal ou Mensal]
        B --> C[Seleciona tipos de atividade\npara contabilizar]
        C --> D[Clica em 'Gerar Relatório']
        D --> Z{Relatório gerado com sucesso?}

        Z -->|Sem atividades no período| E[Visualiza estado vazio\n'Nenhuma atividade registrada']
        Z -->|Com dados| F[Visualiza tabela de membros\ncom total de atividades e ranking]

        F --> G{Deseja exportar?}
        G -->|Sim| H[Seleciona formato: CSV ou PDF]
        H --> I[Faz download do arquivo]
        I --> J{Deseja enviar mensagem?}
        G -->|Não| J

        J -->|Sim| K[Seleciona destinatários\nTodos ou selecionados]
        K --> L[Redige mensagem de motivação]
        L --> M[Clica em 'Enviar']
        M --> N[Visualiza confirmação:\n'Mensagem enviada para X membros']
        J -->|Não| O([Fim])
        N --> O
        E --> O
        I -->|sem mensagem| O
    end

    subgraph Sistema["⚙️ Sistema"]
        D2[Valida academia_id\ndo token de sessão]
        D2 --> D3[Consulta atividades do período\nFiltra por academia e tipos selecionados\n— SUB2 como fonte de dados]
        D3 --> D4{Há atividades?}
        D4 -->|Não| D5[Retorna resposta vazia]
        D4 -->|Sim| D6[Agrega por atleta:\ntotal_atividades, total_km, total_kcal]
        D6 --> D7[Calcula ranking por\ntotal de atividades no período]
        D7 --> D8[Persiste Relatorio e\nItemRelatorio no banco]
        D8 --> D9[Retorna dados do relatório]

        D5 --> resp_vazia([resposta vazia])
        D9 --> resp_dados([dados do relatório])

        M2[Valida token e academia_id]
        M2 --> M3[Verifica membros ativos]
        M3 --> M4[Envia notificação push\npara cada destinatário via ServicoNotificacao]
        M4 --> M5[Persiste Notificacao no banco]
        M5 --> M6[Retorna contagem de enviados]
    end

    %% Conexões entre swimlanes
    D -->|POST /relatorios| D2
    D5 --> Z
    D9 --> Z
    H -->|GET /relatorios/{id}/export?formato=csv| export_proc[Gera arquivo no backend]
    export_proc --> I
    M -->|POST /notificacoes/academia/{id}| M2
    M6 --> N
```

---

## Notas sobre o diagrama

Isolamento por academia (tenant): o primeiro passo do Sistema, antes de qualquer consulta, é validar que o `academia_id` do token de sessão corresponde ao escopo da requisição. Isso implementa o requisito de isolamento de dados por academia (NF-SUB1-004). Um administrador não pode, mesmo construindo manualmente a requisição, visualizar dados de outra academia.

Fronteira SUB1 ↔ SUB2: a consulta de atividades no passo `D3` atravessa a fronteira entre subsistemas: SUB1 (que gera o relatório) consome dados do SUB2 (histórico de atividades dos atletas). No código, isso é representado por uma chamada de serviço entre módulos com interface bem definida — não por acesso direto ao banco de dados do SUB2 a partir do SUB1.

Paralelismo implícito: exportação e envio de mensagem são ações independentes. Um administrador pode exportar sem enviar mensagem, ou enviar sem exportar. O diagrama representa isso com os dois ramos convergindo para o nó de decisão `{Deseja enviar mensagem?}`.

Persistência do relatório: o Sistema persiste o relatório gerado (`Relatorio` + `ItemRelatorio`) antes de retornar os dados. Isso permite que o administrador acesse relatórios anteriores sem re-processamento, alinhado com a decisão de design descrita na Seção 2 (divergências MER × diagrama de classes).

Geração do arquivo de exportação: a exportação (CSV/PDF) é uma ação do Sistema sobre o relatório já persistido, não uma re-consulta ao banco de atividades. Isso garante que o arquivo exportado reflita o snapshot do momento da geração, não os dados atuais (que podem ter mudado com novos registros).
