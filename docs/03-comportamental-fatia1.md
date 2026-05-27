# Seção 3 — Modelagem Comportamental: Fatia 1

## Fatia 1 — Atleta registra atividade física com cálculo de calorias

**Opção A — Diagrama de Sequência**

Justificativa: A Fatia 1 envolve coordenação entre múltiplos componentes numa sequência temporal bem definida — Atleta (ator externo), Interface (frontend), Sistema de Registro (backend/SUB2) e Calculadora de Calorias (componente interno com regra de negócio). O diagrama de sequência é o mais adequado porque: (a) há mensagens síncronas com retorno entre camadas distintas; (b) a lógica condicional (`alt`) é necessária para modelar os caminhos de erro (dados físicos ausentes, campos inválidos por tipo de atividade) e o cálculo alternativo por categoria; (c) o tempo de execução é linear e curto, sem ciclo de vida duradouro que justificasse um diagrama de estados.

---

## Diagrama de Sequência

```mermaid
sequenceDiagram
    autonumber
    actor Atleta
    participant UI as Interface (Mobile/Web)
    participant API as Sistema (SUB2 - API)
    participant DB as Banco de Dados
    participant Calc as CalculadoraCalorias

    Atleta->>UI: Abre tela "Registrar Atividade"
    UI->>API: GET /atleta/{id}/perfil
    API->>DB: SELECT peso_kg, altura_cm WHERE atleta_id = {id}
    DB-->>API: dados físicos (pode ser null)

    alt dados físicos ausentes
        API-->>UI: 200 OK + aviso: "Informe peso e altura para cálculo de calorias"
        UI-->>Atleta: Exibe formulário com banner de aviso
    else dados físicos presentes
        API-->>UI: 200 OK + {peso, altura}
        UI-->>Atleta: Exibe formulário sem aviso
    end

    Atleta->>UI: Preenche tipo, duração e distância (opcional)
    Atleta->>UI: Clica "Salvar"

    UI->>API: POST /atividades {tipo, duracaoMinutos, distanciaKm, atletaId}

    API->>API: validarCampos(atividade)

    alt campos inválidos
        note right of API: duração ≤ 0, ou distância ausente para cardio
        API-->>UI: 400 Bad Request + mensagem de erro
        UI-->>Atleta: Exibe erros de validação
    else campos válidos

        alt tipo = MUSCULACAO ou OUTRO
            API->>Calc: calcular(atividade, atleta) via CalculadoraMusculacao
        else tipo = CAMINHADA, CORRIDA ou CICLISMO
            API->>Calc: calcular(atividade, atleta) via CalculadoraCardio
        end

        Calc-->>API: caloriasEstimadas (Double)

        alt dados físicos ausentes
            note right of API: caloriasEstimadas = null
        end

        API->>DB: INSERT INTO atividade (atletaId, tipo, duracao, distancia, calorias, realizadaEm)
        DB-->>API: atividade persistida com id

        API->>DB: UPDATE meta SET progresso_atual WHERE ativa AND tipo compatível
        DB-->>API: metas atualizadas

        API-->>UI: 201 Created + {atividade, caloriasEstimadas, metasAtualizadas}
        UI-->>Atleta: Exibe confirmação com calorias estimadas
        note over UI,Atleta: Se calorias presentes: "~{X} kcal estimadas*"\nSe ausentes: "Informe peso/altura para ver calorias"
    end
```

---

## Notas sobre o diagrama

Fragmento `alt` — dados físicos ausentes: o sistema consulta os dados físicos antes de exibir o formulário para personalizar a experiência. Se ausentes, exibe aviso sem bloquear o registro — o atleta pode registrar atividades sem informar peso e altura, mas não terá o cálculo de calorias.

Fragmento `alt` — validação: a validação acontece no backend após o POST. Campos obrigatórios variam por tipo: distância é obrigatória para cardio (caminhada, corrida, ciclismo) e opcional para musculação.

Fragmento `alt` — seleção da calculadora: implementação do padrão Strategy modelado no diagrama de classes. A API seleciona a implementação correta de `CalculadoraCalorias` com base no tipo de atividade.

Atualização de metas: ao persistir a atividade, o sistema verifica se há metas ativas compatíveis com o tipo e a métrica da atividade registrada e atualiza o progresso. Isso é um efeito colateral do registro, não uma ação separada do atleta.

Nota de estimativa: a interface deve deixar explícito que o valor exibido é uma estimativa (`*`), conforme critério de aceitação da US-SUB2-002 e RF05 do documento de visão.
