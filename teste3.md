
```mermaid
classDiagram

%% ====================== MODEL ======================

    class User {
        - id: number
        - nickname: string
        - picture: Buffer
        - teamId: number
        - roleKindId: number
        + getTeam() Team
        + getRole() RoleKind
    }

    class UserSensitive {
        - id: number
        - userId: number
        - name: string
        - email: string
        - document: string
        + getUser() User
    }

    class Team {
        - id: number
        - name: string
        - color: string
        - captainId: number
        - analystId: number
        + getTotalKm() number
        + getMembers() User[]
    }

    class RunningBelt {
        - id: number
        - teamId: number
        - name: string
        - statusKindId: number
        + getStatus() StatusKind
        + getLastRecord() RunRecord
    }

    class RunRecord {
        - id: number
        - runningBeltId: number
        - userId: number
        - kmInicial: number
        - kmFinal: number
        - epochTime: Date
        - isOpen: boolean
        - synced: boolean
        + getKmIncrement() number
        + close() void
    }

    class Audit {
        - id: number
        - userId: number
        - auditKindId: number
        - epochTime: number
        - auditInfo: object
        + getInfo() string
    }

    class SyncQueue {
        - id: number
        - runRecordId: number
        - payload: object
        - createdAt: Date
        - sent: boolean
        + markAsSent() void
    }

    class Statistic {
        - id: number
        - userId: number
        - teamId: number
        - statsKindId: number
        - value: number
        - lastSync: Date
        + getValue() number
    }

    class Report {
        - id: number
        - generatedBy: number
        - format: string
        - encoding: string
        - createdAt: Date
        + getGeneratedBy() User
    }

    class RoleKind {
        <<enumeration>>
        RUNNER
        ANALYST
        ADMIN
    }

    class StatusKind {
        <<enumeration>>
        FREE
        IN_USE
        MAINTENANCE
    }

    class AuditKind {
        <<enumeration>>
        RUN_START
        RUN_END
        CHECKPOINT
        CORRECTION
    }

    class StatsKind {
        <<enumeration>>
        KILOMETERS
        CHECKPOINT_30MIN
        TOTAL_TIME
    }

    User --> RoleKind
    User "1" *-- "1" UserSensitive
    Team "1" *-- "2" RunningBelt
    RunningBelt --> StatusKind
    RunRecord "0..*" --> "1" User
    RunRecord "0..*" --> "1" RunningBelt
    Audit --> AuditKind
    Statistic --> StatsKind

%% ==================== REPOSITORY ====================

    class IRunRecordRepository {
        <<interface>>
        + findById(id: number) Promise~RunRecord~
        + findByEsteira(esteiraId: number) Promise~RunRecord[]~
        + findUltimoKmFinal(esteiraId: number) Promise~number~
        + findAbertos() Promise~RunRecord[]~
        + save(record: RunRecord) Promise~RunRecord~
        + update(id: number, data: object) Promise~RunRecord~
    }

    class IUserRepository {
        <<interface>>
        + findById(id: number) Promise~User~
        + findByNickname(nickname: string) Promise~User~
        + findByTeam(teamId: number) Promise~User[]~
        + findCorridaAberta(userId: number) Promise~RunRecord~
        + findSensitive(userId: number) Promise~UserSensitive~
        + save(user: User) Promise~User~
        + saveSensitive(data: UserSensitive) Promise~UserSensitive~
    }

    class ITeamRepository {
        <<interface>>
        + findById(id: number) Promise~Team~
        + findAll() Promise~Team[]~
        + getTotalKm(teamId: number) Promise~number~
    }

    class IAuditRepository {
        <<interface>>
        + save(audit: Audit) Promise~Audit~
        + findByUser(userId: number) Promise~Audit[]~
        + findByTipo(kind: AuditKind) Promise~Audit[]~
        + findInconsistencias() Promise~Audit[]~
        + findAll() Promise~Audit[]~
    }

    class ISyncQueueRepository {
        <<interface>>
        + findPendentes() Promise~SyncQueue[]~
        + save(item: SyncQueue) Promise~SyncQueue~
        + markAsSent(id: number) Promise~void~
        + deleteSent() Promise~void~
    }

    class IStatisticRepository {
        <<interface>>
        + findByUser(userId: number) Promise~Statistic[]~
        + findByTeam(teamId: number) Promise~Statistic[]~
        + upsert(data: Statistic) Promise~Statistic~
    }

    class IReportRepository {
        <<interface>>
        + save(report: Report) Promise~Report~
        + findByUser(userId: number) Promise~Report[]~
    }

    class IValidacaoService {
        <<interface>>
        + validarEsteira(esteiraId: number) Promise~boolean~
        + validarParticipante(userId: number) Promise~boolean~
        + validarKm(km: number, esteiraId: number) Promise~boolean~
        + detectarInconsistencia(record: RunRecord) Promise~boolean~
    }

    class RunRecordRepository {
        - db: DataSource
        + findById(id: number) Promise~RunRecord~
        + findByEsteira(esteiraId: number) Promise~RunRecord[]~
        + findUltimoKmFinal(esteiraId: number) Promise~number~
        + findAbertos() Promise~RunRecord[]~
        + save(record: RunRecord) Promise~RunRecord~
        + update(id: number, data: object) Promise~RunRecord~
    }

    class UserRepository {
        - db: DataSource
        + findById(id: number) Promise~User~
        + findByNickname(nickname: string) Promise~User~
        + findByTeam(teamId: number) Promise~User[]~
        + findCorridaAberta(userId: number) Promise~RunRecord~
        + findSensitive(userId: number) Promise~UserSensitive~
        + save(user: User) Promise~User~
        + saveSensitive(data: UserSensitive) Promise~UserSensitive~
    }

    class TeamRepository {
        - db: DataSource
        + findById(id: number) Promise~Team~
        + findAll() Promise~Team[]~
        + getTotalKm(teamId: number) Promise~number~
    }

    class AuditRepository {
        - db: DataSource
        + save(audit: Audit) Promise~Audit~
        + findByUser(userId: number) Promise~Audit[]~
        + findByTipo(kind: AuditKind) Promise~Audit[]~
        + findInconsistencias() Promise~Audit[]~
        + findAll() Promise~Audit[]~
    }

    class SyncQueueRepository {
        - db: DataSource
        + findPendentes() Promise~SyncQueue[]~
        + save(item: SyncQueue) Promise~SyncQueue~
        + markAsSent(id: number) Promise~void~
        + deleteSent() Promise~void~
    }

    class StatisticRepository {
        - db: DataSource
        + findByUser(userId: number) Promise~Statistic[]~
        + findByTeam(teamId: number) Promise~Statistic[]~
        + upsert(data: Statistic) Promise~Statistic~
    }

    class ReportRepository {
        - db: DataSource
        + save(report: Report) Promise~Report~
        + findByUser(userId: number) Promise~Report[]~
    }

    RunRecordRepository ..|> IRunRecordRepository
    UserRepository ..|> IUserRepository
    TeamRepository ..|> ITeamRepository
    AuditRepository ..|> IAuditRepository
    SyncQueueRepository ..|> ISyncQueueRepository
    StatisticRepository ..|> IStatisticRepository
    ReportRepository ..|> IReportRepository

    RunRecordRepository ..> RunRecord
    UserRepository ..> User
    UserRepository ..> UserSensitive
    TeamRepository ..> Team
    AuditRepository ..> Audit
    SyncQueueRepository ..> SyncQueue
    StatisticRepository ..> Statistic
    ReportRepository ..> Report

%% ===================== SERVICE =====================

    class RegistroService {
        - runRecordRepo: IRunRecordRepository
        - userRepo: IUserRepository
        - auditRepo: IAuditRepository
        - syncRepo: ISyncQueueRepository
        - validacaoService: IValidacaoService
        + iniciarTurno(dto: IniciarTurnoDTO) Promise~RunRecord~
        + registrarCheckpoint(dto: CheckpointDTO) Promise~RunRecord~
        + encerrarTurno(id: number, kmFinal: number) Promise~RunRecord~
        + editarRegistro(id: number, dto: EditarRegistroDTO) Promise~RunRecord~
        + sincronizar() Promise~void~
        - calcularKmIncremental(record: RunRecord) number
    }

    class ValidacaoService {
        - runRecordRepo: IRunRecordRepository
        - userRepo: IUserRepository
        - auditRepo: IAuditRepository
        + validarEsteira(esteiraId: number) Promise~boolean~
        + validarParticipante(userId: number) Promise~boolean~
        + validarKm(km: number, esteiraId: number) Promise~boolean~
        + detectarInconsistencia(record: RunRecord) Promise~boolean~
    }

    class EstatisticaService {
        - teamRepo: ITeamRepository
        - runRecordRepo: IRunRecordRepository
        - statisticRepo: IStatisticRepository
        - reportRepo: IReportRepository
        + calcularPlacar() Promise~PlacarDTO~
        + atualizarEstatisticas(record: RunRecord) Promise~void~
        + getContribIndividual(userId: number) Promise~number~
        + getHistoricoEsteira(esteiraId: number) Promise~RunRecord[]~
        + exportarCSV(generatedBy: number) Promise~string~
    }

    class AuditoriaService {
        - auditRepo: IAuditRepository
        + findAll() Promise~Audit[]~
        + findInconsistencias() Promise~Audit[]~
        + findByUser(userId: number) Promise~Audit[]~
    }

    class AdminService {
        - runRecordRepo: IRunRecordRepository
        - userRepo: IUserRepository
        + listarTodosRegistros() Promise~RunRecord[]~
        + listarFiscais() Promise~User[]~
    }

    ValidacaoService ..|> IValidacaoService
    RegistroService ..> IRunRecordRepository
    RegistroService ..> IUserRepository
    RegistroService ..> IAuditRepository
    RegistroService ..> ISyncQueueRepository
    RegistroService --> IValidacaoService
    ValidacaoService ..> IRunRecordRepository
    ValidacaoService ..> IUserRepository
    ValidacaoService ..> IAuditRepository
    EstatisticaService ..> ITeamRepository
    EstatisticaService ..> IRunRecordRepository
    EstatisticaService ..> IStatisticRepository
    EstatisticaService ..> IReportRepository
    AuditoriaService ..> IAuditRepository
    AdminService ..> IRunRecordRepository
    AdminService ..> IUserRepository

%% ==================== CONTROLLER ===================

    class RegistroController {
        - registroService: RegistroService
        + POST /turnos iniciarTurno(req, res)
        + POST /turnos/:id/checkpoints registrarCheckpoint(req, res)
        + POST /turnos/:id/encerramento encerrarTurno(req, res)
        + PUT /registros/:id editarRegistro(req, res)
        + POST /registros/sincronizar sincronizar(req, res)
    }

    class AdminController {
        - estatisticaService: EstatisticaService
        - auditoriaService: AuditoriaService
        - adminService: AdminService
        + GET /admin/exportar exportarCSV(req, res)
        + GET /admin/registros listarRegistros(req, res)
        + GET /admin/fiscais listarFiscais(req, res)
        + GET /admin/auditoria getAuditoria(req, res)
    }

    class PlacarController {
        - estatisticaService: EstatisticaService
        + GET /placar/equipes getPlacarEquipes(req, res)
        + GET /esteiras/:id/status getStatusEsteiras(req, res)
        + GET /esteiras/:id/registros getHistoricoEsteira(req, res)
    }

    RegistroController --> RegistroService
    AdminController --> EstatisticaService
    AdminController --> AuditoriaService
    AdminController --> AdminService
    PlacarController --> EstatisticaService

%% ==================== DTO ======================

    class IniciarTurnoDTO {
        + equipeId: number
        + esteiraId: number
        + userId: number
    }

    class CheckpointDTO {
        + esteiraId: number
        + kmAtual: number
    }

    class EditarRegistroDTO {
        + kmFinal: number
        + justificativa: string
    }

    class PlacarDTO {
        + equipeA: number
        + equipeB: number
        + atualizadoEm: Date
    }

    RegistroController ..> IniciarTurnoDTO
    RegistroController ..> CheckpointDTO
    RegistroController ..> EditarRegistroDTO
    PlacarController ..> PlacarDTO
    RegistroService ..> IniciarTurnoDTO
    RegistroService ..> CheckpointDTO
    RegistroService ..> EditarRegistroDTO
    EstatisticaService ..> PlacarDTO
```


