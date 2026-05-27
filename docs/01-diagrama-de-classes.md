# Seção 1 — Diagrama de Classes

O diagrama abaixo modela as classes envolvidas nas três fatias selecionadas. Classes que aparecem em mais de uma fatia são representadas uma única vez.

---

## 1 Diagrama

```mermaid
classDiagram
    direction TB

    %% ─── Usuários ───────────────────────────────────────────────────
    class Usuario {
        <<abstract>>
        +Long id
        +String nome
        +String email
        +String senhaHash
        +DateTime criadoEm
        +autenticar(email, senha) Boolean
        +atualizarEmail(novoEmail) void
    }

    class Atleta {
        +Double pesoKg
        +Double alturaCm
        +DateTime dataNascimento
        +atualizarDadosFisicos(peso, altura) void
        +calcularIMC() Double
    }

    class AdministradorAcademia {
        +enviarMensagem(texto, destinatarios) void
        +gerarRelatorio(periodo, tiposAtividade) Relatorio
        +encerrarDesafio(desafio) void
    }

    Usuario <|-- Atleta : herança
    Usuario <|-- AdministradorAcademia : herança

    %% ─── Academia ────────────────────────────────────────────────────
    class Academia {
        +Long id
        +String nome
        +String endereco
        +String tipo
        +DateTime criadaEm
        +adicionarMembro(atleta) void
        +removerMembro(atleta) void
        +listarMembrosAtivos() List~Atleta~
    }

    AdministradorAcademia "1" --> "1" Academia : administra
    Academia "1" o-- "0..*" Atleta : membros

    %% ─── Atividade ──────────────────────────────────────────────────
    class TipoAtividade {
        <<enumeration>>
        CAMINHADA
        CORRIDA
        CICLISMO
        MUSCULACAO
        OUTRO
    }

    class Atividade {
        +Long id
        +TipoAtividade tipo
        +Integer duracaoMinutos
        +Double distanciaKm
        +Double caloriasEstimadas
        +DateTime realizadaEm
        +Boolean importadaDeDispositivo
        +calcularCalorias(atleta) Double
        +validarCampos() void
    }

    class CalculadoraCalorias {
        <<interface>>
        +calcular(atividade, atleta) Double
    }

    class CalculadoraCardio {
        +calcular(atividade, atleta) Double
    }

    class CalculadoraMusculacao {
        +calcular(atividade, atleta) Double
    }

    CalculadoraCalorias <|.. CalculadoraCardio : realização
    CalculadoraCalorias <|.. CalculadoraMusculacao : realização

    Atleta "1" *-- "0..*" Atividade : histórico
    Atividade --> TipoAtividade : usa
    Atividade ..> CalculadoraCalorias : <<depende>>

    %% ─── Meta ───────────────────────────────────────────────────────
    class TipoMeta {
        <<enumeration>>
        DISTANCIA
        CALORIAS
        TEMPO
    }

    class Meta {
        +Long id
        +TipoMeta tipo
        +Double valorAlvo
        +Double progressoAtual
        +DateTime prazo
        +Boolean concluida
        +atualizarProgresso(atividade) void
        +verificarConclusao() Boolean
    }

    Atleta "1" *-- "0..*" Meta : define
    Meta --> TipoMeta : usa

    %% ─── Desafio ────────────────────────────────────────────────────
    class StatusDesafio {
        <<enumeration>>
        RASCUNHO
        ATIVO
        ENCERRADO
        ARQUIVADO
    }

    class Desafio {
        +Long id
        +String nome
        +String descricao
        +TipoMeta metrica
        +Double valorMeta
        +DateTime inicio
        +DateTime prazo
        +StatusDesafio status
        +Boolean apenasAcademia
        +ativar() void
        +encerrar() void
        +arquivar() void
        +calcularRanking() List~InscricaoDesafio~
    }

    class InscricaoDesafio {
        +Long id
        +Double progressoAcumulado
        +Integer posicaoRanking
        +DateTime inscritoEm
        +atualizarProgresso(atividade) void
        +calcularPosicao() Integer
    }

    Academia "1" o-- "0..*" Desafio : promove
    Atleta "1" *-- "0..*" InscricaoDesafio : participa
    Desafio "1" *-- "0..*" InscricaoDesafio : inscrições
    Desafio --> StatusDesafio : usa
    Desafio --> TipoMeta : usa

    %% ─── Relatório ──────────────────────────────────────────────────
    class PeriodoRelatorio {
        <<enumeration>>
        SEMANAL
        MENSAL
    }

    class Relatorio {
        +Long id
        +PeriodoRelatorio periodo
        +DateTime geradoEm
        +List~ItemRelatorio~ itens
        +exportarCSV() String
        +exportarPDF() Byte[]
    }

    class ItemRelatorio {
        +Atleta atleta
        +Integer totalAtividades
        +Double totalDistanciaKm
        +Double totalCaloriasEstimadas
        +Integer posicaoRanking
    }

    Relatorio "1" *-- "1..*" ItemRelatorio : composto
    ItemRelatorio --> Atleta : referencia

    %% ─── Notificação ────────────────────────────────────────────────
    class Notificacao {
        +Long id
        +String titulo
        +String corpo
        +DateTime enviadaEm
        +Boolean lida
    }

    class ServicoNotificacao {
        <<interface>>
        +enviarPush(usuario, notificacao) void
        +enviarParaGrupo(atletas, notificacao) void
    }

    Atleta "1" *-- "0..*" Notificacao : recebe
    AdministradorAcademia ..> ServicoNotificacao : <<depende>>
    Desafio ..> ServicoNotificacao : <<depende>>
```

---

## 1 Conceitos de UML

Hierarquia de `Usuario`: optou-se por uma classe abstrata `Usuario` com subclasses `Atleta` e `AdministradorAcademia`. A alternativa de uma única classe com atributo `perfil` (enum) foi descartada porque os dois perfis possuem atributos exclusivos (`pesoKg`, `alturaCm` para o atleta; vínculo com `Academia` para o administrador) e comportamentos claramente distintos, herança é mais expressiva neste caso.

Interface `CalculadoraCalorias`: a fórmula de cálculo varia por tipo de atividade (cardio usa duração × MET × peso, musculação usa uma estimativa diferente). Modelar com uma interface e implementações concretas por categoria permite extensão sem modificar a classe `Atividade`. Esta é uma aplicação do padrão Strategy.

`InscricaoDesafio` como classe associativa: o relacionamento entre `Atleta` e `Desafio` é muitos-para-muitos e carrega estado próprio (`progressoAcumulado`, `posicaoRanking`, `inscritoEm`). Uma tabela associativa simples não bastaria, a classe associativa é a representação correta.

`Relatorio` e `ItemRelatorio`: o relatório é um artefato gerado sob demanda (não é dado primário), mas pode ser persistido como cache para exportação. `ItemRelatorio` é uma composição, não existe sem o `Relatorio` que o originou.

`ServicoNotificacao` como interface: o serviço de envio de notificações é uma integração externa (push, e-mail). Modelar como interface permite que a implementação concreta seja trocada sem afetar o domínio, e explicita a dependência externa sem hard-codar detalhes de infraestrutura no diagrama de classes.

---

## 1.2 Classes por fatia

| Fatia | Classes envolvidas |
|---|---|
| Fatia 1 — Registro com calorias | `Atleta`, `Atividade`, `TipoAtividade`, `CalculadoraCalorias`, `CalculadoraCardio`, `CalculadoraMusculacao` |
| Fatia 2 — Ciclo de vida do desafio | `Atleta`, `AdministradorAcademia`, `Academia`, `Desafio`, `InscricaoDesafio`, `StatusDesafio`, `ServicoNotificacao` |
| Fatia 3 — Relatório do administrador | `AdministradorAcademia`, `Academia`, `Atleta`, `Atividade`, `Relatorio`, `ItemRelatorio`, `ServicoNotificacao` |