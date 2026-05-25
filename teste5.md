```mermaid
classDiagram

%% ====================== MODEL ======================

    class User {
        - id: number
        - name: string
        - email: string
        - document: string
        - nickname: string
        - teamId: number
        - role: RoleKind
    }

    class Team {
        - id: number
        - name: string
        - color: string
        - captainId: number
        + getTotalKm() number
    }

    class RunningBelt {
        - id: number
        - teamId: number
        - name: string
        - status: StatusKind
    }

    class RunRecord {
        - id: number
        - runningBeltId: number
        - userId: number
        - startTime: Date
        - endTime: Date
        - isFinished: boolean
        - isSynced: boolean
    }

    class Checkpoint {
        - id: number
        - runRecordId: number
        - currentKm: number
        - registeredAt: Date
        - isSynced: boolean
    }

    class AuditLog {
        - id: number
        - userId: number
        - action: string
        - timestamp: Date
        - details: string
    }

    class RoleKind {
        <<enumeration>>
        RUNNER
        FISCAL
        ADMIN
    }

    class StatusKind {
        <<enumeration>>
        FREE
        IN_USE
        MAINTENANCE
    }

    User --> RoleKind : possui
    User "0..*" --> "0..1" Team : pertence a
    Team "1" *-- "2" RunningBelt : possui
    Team "0..*" --> "1" User : tem capitão
    RunningBelt --> StatusKind : possui
    RunningBelt "1" o-- "0..*" RunRecord : registra
    User "1" o-- "0..*" RunRecord : corre em
    RunRecord "1" *-- "1..*" Checkpoint : possui historico
    User "1" o-- "0..*" AuditLog : gera

%% ==================== REPOSITORY ====================

    class UserRepository {
        + findById(id: number) User
        + findByTeam(teamId: number) User[]
        + save(user: User) User
    }

    class TeamRepository {
        + findById(id: number) Team
        + findAll() Team[]
    }

    class RunRecordRepository {
        + findById(id: number) RunRecord
        + findActiveRecord(beltId: number) RunRecord
        + save(record: RunRecord) RunRecord
        + update(record: RunRecord) RunRecord
    }

    class CheckpointRepository {
        + save(checkpoint: Checkpoint) Checkpoint
        + findByRunRecord(runRecordId: number) Checkpoint[]
        + findLastByBelt(beltId: number) Checkpoint
    }

    class AuditLogRepository {
        + save(log: AuditLog) AuditLog
        + findAll() AuditLog[]
    }

    RunRecordRepository ..> RunRecord : gerencia
    CheckpointRepository ..> Checkpoint : gerencia
    UserRepository ..> User : gerencia
    TeamRepository ..> Team : gerencia
    AuditLogRepository ..> AuditLog : gerencia

%% ===================== SERVICE =====================

    class RegistroService {
        - runRecordRepo: RunRecordRepository
        - checkpointRepo: CheckpointRepository
        - auditRepo: AuditLogRepository
        + iniciarTurno(userId: number, beltId: number) RunRecord
        + registrarCheckpoint(runRecordId: number, km: number) Checkpoint
        + encerrarTurno(runRecordId: number, kmFinal: number) RunRecord
        + sincronizarDadosOffline(dadosPendentes: object) void
    }

    class EstatisticaService {
        - teamRepo: TeamRepository
        - runRecordRepo: RunRecordRepository
        - checkpointRepo: CheckpointRepository
        + calcularPlacarEquipes() object
        + exportarHistoricoCSV() string
    }

    class ValidacaoService {
        - checkpointRepo: CheckpointRepository
        + validarNovoKm(novoKm: number, beltId: number) boolean
    }

    RegistroService ..> RunRecordRepository : usa
    RegistroService ..> CheckpointRepository : usa
    RegistroService ..> AuditLogRepository : usa
    RegistroService --> ValidacaoService : valida regras
    EstatisticaService ..> TeamRepository : usa
    EstatisticaService ..> CheckpointRepository : calcula km

%% ==================== CONTROLLER ===================

    class RegistroController {
        - registroService: RegistroService
        + POST /turnos/iniciar
        + POST /turnos/:id/checkpoints
        + POST /turnos/:id/encerrar
        + POST /sincronizar
    }

    class PainelController {
        - estatisticaService: EstatisticaService
        + GET /placar
        + GET /exportar-csv
    }

    RegistroController --> RegistroService : chama
    PainelController --> EstatisticaService : chama