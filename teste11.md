```mermaid
classDiagram
    direction LR

    %% ==========================================
    %% CAMADA DE INTERFACE (WEB)
    %% ==========================================
    class RegistroController {
        + iniciarTurno(req, res): void
        + registrarCheckpoint(req, res): void
        + encerrarTurno(req, res): void
    }

    class PainelController {
        + obterPlacar(req, res): void
    }

    class RelatorioController {
        + gerarRelatorio(req, res): void
    }

    class UsuarioController {
        + obterPerfil(req, res): void
    }

    %% ==========================================
    %% CAMADA DE NEGÓCIO (SERVICES)
    %% ==========================================
    class RegistroService {
        + iniciarTurno(userId, runningBeltId, kmInicial): void
        + registrarCheckpoint(runRecordId, kmAtual): void
        + encerrarTurno(runRecordId, kmFinal): void
        + processarFilaOffline(): void
    }

    class EstatisticaService {
        + consolidarPlacar(): object
    }

    class RelatorioService {
        + criarRelatorioFormatado(generatedBy, format, encoding): object
    }

    class UsuarioService {
        + obterDadosCompletos(userId): object
    }

    %% ==========================================
    %% OPERAÇÕES DIRETAS NAS TABELAS DO SEU DER
    %% ==========================================
    class USERS {
        + id: int
        + nickname: varchar
        + picture: bytea
        + team_id: int
        + role_kind_id: int
        + created_at: timestamp
    }

    class USER_SENSITIVE {
        + id: int
        + user_id: int
        + name: varchar
        + email: varchar
        + document: varchar
    }

    class TEAMS {
        + id: int
        + name: varchar
        + color: varchar
        + captain_id: int
        + analist_id: int
    }

    class RUNNING_BELT {
        + id: int
        + team_id: int
        + name: varchar
        + status_kind_id: int
    }

    class RUN_RECORD {
        + id: int
        + running_belt_id: int
        + user_id: int
        + km_inicial: decimal
        + km_final: decimal
        + epoch_time: timestamp
        + is_open: boolean
        + synced: boolean
    }

    class SYNC_QUEUE {
        + id: int
        + run_record_id: int
        + payload: json
        + created_at: timestamp
        + sent: boolean
    }

    class AUDIT {
        + id: int
        + user_id: int
        + audit_kind_id: int
        + epoch_time: bigint
        + audit_info: json
    }

    class STATISTICS {
        + id: int
        + user_id: int
        + team_id: int
        + stats_kind_id: int
        + value: decimal
        + last_sync: timestamp
    }

    class REPORT {
        + id: int
        + generated_by: int
        + format: varchar
        + encoding: varchar
        + created_at: timestamp
    }

    %% ==========================================
    %% RELACIONAMENTOS DIRETOS
    %% ==========================================
    RegistroController --> RegistroService : aciona
    PainelController --> EstatisticaService : aciona
    RelatorioController --> RelatorioService : aciona
    UsuarioController --> UsuarioService : aciona

    RegistroService --> RUN_RECORD : manipula
    RegistroService --> RUNNING_BELT : atualiza status
    RegistroService --> SYNC_QUEUE : enfileira
    RegistroService --> AUDIT : insere log
    RegistroService --> STATISTICS : incrementa valor

    EstatisticaService --> STATISTICS : lê consolidados
    EstatisticaService --> TEAMS : agrupa dados

    RelatorioService --> REPORT : salva
    RelatorioService --> STATISTICS : extrai métricas

    UsuarioService --> USERS : busca público
    UsuarioService --> USER_SENSITIVE : busca privado