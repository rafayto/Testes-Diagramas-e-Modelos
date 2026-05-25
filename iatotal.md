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

    User --> RoleKind
    User "0..*" --> "0..1" Team
    Team "1" *-- "2" RunningBelt
    Team "0..*" --> "1" User
    RunningBelt --> StatusKind
    RunningBelt "1" o-- "0..*" RunRecord
    User "1" o-- "0..*" RunRecord
    RunRecord "1" *-- "1..*" Checkpoint
    User "1" o-- "0..*" AuditLog

%% ==================== REPOSITORY ====================
    class IUserRepository {
        <<interface>>
        + findById(id: number) Promise~User~
        + findByTeam(teamId: number) Promise~User[]~
        + save(user: User) Promise~User~
    }

    class ITeamRepository {
        <<interface>>
        + findById(id: number) Promise~Team~
        + findAll() Promise~Team[]~
    }

    class IRunRecordRepository {
        <<interface>>
        + findById(id: number) Promise~RunRecord~
        + findActiveRecord(beltId: number) Promise~RunRecord~
        + save(record: RunRecord) Promise~RunRecord~
    }

    class ICheckpointRepository {
        <<interface>>
        + save(checkpoint: Checkpoint) Promise~Checkpoint~
        + findByRunRecord(runRecordId: number) Promise~Checkpoint[]~
        + findLastByBelt(beltId: number) Promise~Checkpoint~
    }

    class IAuditLogRepository {
        <<interface>>
        + save(log: AuditLog) Promise~AuditLog~
        + findAll() Promise~AuditLog[]~
    }

    class UserRepository {
        - orm: DataSource
        + findById(id)
        + findByTeam(teamId)
        + save(user)
    }

    class TeamRepository {
        - orm: DataSource
        + findById(id)
        + findAll()
    }

    class RunRecordRepository {
        - orm: DataSource
        + findById(id)
        + findActiveRecord(beltId)
        + save(record)
    }

    class CheckpointRepository {
        - orm: DataSource
        + save(checkpoint)
        + findByRunRecord(runRecordId)
        + findLastByBelt(beltId)
    }

    class AuditLogRepository {
        - orm: DataSource
        + save(log)
        + findAll()
    }

    UserRepository ..|> IUserRepository
    TeamRepository ..|> ITeamRepository
    RunRecordRepository ..|> IRunRecordRepository
    CheckpointRepository ..|> ICheckpointRepository
    AuditLogRepository ..|> IAuditLogRepository

    UserRepository ..> User
    TeamRepository ..> Team
    RunRecordRepository ..> RunRecord
    CheckpointRepository ..> Checkpoint
    AuditLogRepository ..> AuditLog

%% ===================== SERVICE =====================
    class RegistroService {
        - runRecordRepo: IRunRecordRepository
        - checkpointRepo: ICheckpointRepository
        - auditRepo: IAuditLogRepository
        + iniciarTurno(dto: IniciarTurnoDTO) Promise~RunRecord~
        + registrarCheckpoint(dto: RegistrarCheckpointDTO) Promise~Checkpoint~
        + encerrarTurno(dto: EncerrarTurnoDTO) Promise~RunRecord~
        + sincronizarDadosOffline(dadosPendentes: object) Promise~void~
    }

    class EstatisticaService {
        - teamRepo: ITeamRepository
        - runRecordRepo: IRunRecordRepository
        - checkpointRepo: ICheckpointRepository
        + calcularPlacarEquipes() Promise~object~
        + exportarHistoricoCSV() Promise~string~
    }

    class ValidacaoService {
        - checkpointRepo: ICheckpointRepository
        + validarNovoKm(novoKm: number, beltId: number) Promise~boolean~
    }

    RegistroService ..> IRunRecordRepository
    RegistroService ..> ICheckpointRepository
    RegistroService ..> IAuditLogRepository
    RegistroService --> ValidacaoService
    EstatisticaService ..> ITeamRepository
    EstatisticaService ..> ICheckpointRepository

%% ==================== CONTROLLER ===================
    class RegistroController {
        - registroService: RegistroService
        + POST /turnos/iniciar criar(req, res)
        + POST /turnos/:id/checkpoints checkpoint(req, res)
        + POST /turnos/:id/encerrar encerrar(req, res)
        + POST /sincronizar sync(req, res)
    }

    class PainelController {
        - estatisticaService: EstatisticaService
        + GET /placar obterPlacar(req, res)
        + GET /exportar-csv baixarCSV(req, res)
    }

    RegistroController --> RegistroService
    PainelController --> EstatisticaService

%% ==================== DTO (apoio) ==================
    class IniciarTurnoDTO {
        + userId: number
        + beltId: number
    }
    class RegistrarCheckpointDTO {
        + runRecordId: number
        + km: number
    }
    class EncerrarTurnoDTO {
        + runRecordId: number
        + kmFinal: number
    }

    RegistroController ..> IniciarTurnoDTO
    RegistroController ..> RegistrarCheckpointDTO
    RegistroController ..> EncerrarTurnoDTO
    RegistroService ..> IniciarTurnoDTO
    RegistroService ..> RegistrarCheckpointDTO
    RegistroService ..> EncerrarTurnoDTO