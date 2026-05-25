```mermaid
classDiagram

%% ==================== CAMADA: MODEL ====================
    class User {
        - id: number
        - nickname: string
        - teamId: number
        - role: string
    }

    class UserSensitive {
        - id: number
        - userId: number
        - name: string
        - email: string
        - document: string
    }

    class Team {
        - id: number
        - name: string
        - color: string
        - captainId: number
    }

    class RunningBelt {
        - id: number
        - teamId: number
        - status: string
    }

    class RunRecord {
        - id: number
        - runningBeltId: number
        - userId: number
        - kmInicial: number
        - kmFinal: number
        - isOpen: boolean
        - synced: boolean
    }

    class Audit {
        - id: number
        - userId: number
        - auditType: string
        - info: string
    }

    class SyncQueue {
        - id: number
        - runRecordId: number
        - payload: object
        - sent: boolean
    }

    class Statistic {
        - id: number
        - teamId: number
        - statType: string
        - value: number
    }

    class Report {
        - id: number
        - generatedBy: number
        - format: string
        - createdAt: Date
    }

    User "1" *-- "1" UserSensitive : LGPD
    Team "1" *-- "2" RunningBelt
    RunningBelt "1" o-- "0..*" RunRecord
    User "1" o-- "0..*" RunRecord
    RunRecord "1" *-- "0..*" SyncQueue
    User "1" o-- "0..*" Audit
    Team "1" o-- "0..*" Statistic

%% ==================== CAMADA: REPOSITORY ====================
    class UserRepository {
        + findById(id) User
        + findSensitiveData(userId) UserSensitive
    }

    class RunRecordRepository {
        + findActive(beltId) RunRecord
        + save(record) RunRecord
    }

    class SyncQueueRepository {
        + findPending() SyncQueue[]
        + markAsSent(id) void
    }

    class StatisticRepository {
        + updateTeamStats(teamId, value) Statistic
    }

    class ReportRepository {
        + save(report) Report
    }

%% ==================== CAMADA: SERVICE ====================
    class RegistroService {
        - runRecordRepo: RunRecordRepository
        - syncQueueRepo: SyncQueueRepository
        + iniciarTurno(userId, beltId) RunRecord
        + atualizarKm(runRecordId, kmAtual) void
        + processarFilaOffline() void
    }

    class EstatisticaService {
        - statRepo: StatisticRepository
        - reportRepo: ReportRepository
        + consolidarKms() void
        + gerarRelatorioDiario(userId, formato) Report
    }

    RegistroService ..> RunRecordRepository
    RegistroService ..> SyncQueueRepository
    EstatisticaService ..> StatisticRepository
    EstatisticaService ..> ReportRepository

%% ==================== CAMADA: CONTROLLER ===================
    class RegistroController {
        - registroService: RegistroService
        + POST /turnos/iniciar iniciar()
        + PUT /turnos/:id/km atualizarKm()
        + POST /sincronizar syncOffline()
    }

    class PainelController {
        - estatisticaService: EstatisticaService
        + GET /estatisticas obterDados()
        + POST /relatorios gerarRelatorio()
    }

    RegistroController --> RegistroService
    PainelController --> EstatisticaService