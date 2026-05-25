```mermaid
classDiagram
    direction TB

    %% ==========================================
    %% CAMADA DE APRESENTAÇÃO / ORQUESTRAÇÃO (WEB)
    %% ==========================================
    class RegistroController {
        - registroService: RegistroService
        + iniciarTurno(req, res): void
        + registrarCheckpoint(req, res): void
        + encerrarTurno(req, res): void
    }

    class PainelController {
        - estatisticaService: EstatisticaService
        + obterPlacar(req, res): void
    }

    class RelatorioController {
        - relatorioService: RelatorioService
        + gerarRelatorio(req, res): void
    }

    class UsuarioController {
        - usuarioService: UsuarioService
        + obterPerfil(req, res): void
    }

    %% ==========================================
    %% CAMADA DE REGRAS DE NEGÓCIO (DOMAIN)
    %% ==========================================
    class RegistroService {
        - runRecordRepository: RunRecordRepository
        - runningBeltRepository: RunningBeltRepository
        - auditRepository: AuditRepository
        - syncQueueRepository: SyncQueueRepository
        - statisticsRepository: StatisticsRepository
        + iniciarTurno(userId, runningBeltId, kmInicial): RunRecord
        + registrarCheckpoint(runRecordId, kmAtual): void
        + encerrarTurno(runRecordId, kmFinal): void
        + processarFilaOffline(): void
    }

    class EstatisticaService {
        - statisticsRepository: StatisticsRepository
        - teamRepository: TeamRepository
        - userRepository: UserRepository
        + consolidarPlacar(): object
        + atualizarMetricasGlobais(): void
    }

    class RelatorioService {
        - reportRepository: ReportRepository
        - statisticsRepository: StatisticsRepository
        + criarRelatorioFormatado(userId, format, encoding): Report
    }

    class UsuarioService {
        - userRepository: UserRepository
        - userSensitiveRepository: UserSensitiveRepository
        + obterDadosCompletos(userId): object
    }

    %% ==========================================
    %% CAMADA DE PERSISTÊNCIA (INFRASTRUCTURE)
    %% ==========================================
    class UserRepository {
        - db: Database
        + findById(id): User
        + save(user): User
    }

    class UserSensitiveRepository {
        - db: Database
        + findByUserId(userId): UserSensitive
    }

    class TeamRepository {
        - db: Database
        + findById(id): Team
        + findStats(teamId): object
    }

    class RunningBeltRepository {
        - db: Database
        + findById(id): RunningBelt
        + updateStatus(id, statusKindId): void
    }

    class RunRecordRepository {
        - db: Database
        + save(record): RunRecord
        + findActiveByUser(userId): RunRecord
        + closeRecord(id, kmFinal): void
    }

    class SyncQueueRepository {
        - db: Database
        + pushToQueue(payload): void
        + findPending(): SyncQueue[]
        + markAsSent(id): void
    }

    class AuditRepository {
        - db: Database
        + saveLog(userId, auditKindId, info): void
    }

    class StatisticsRepository {
        - db: Database
        + saveOrUpdate(stats): void
        + getAggregatedStats(kindId): object
    }

    class ReportRepository {
        - db: Database
        + save(report): Report
    }

    %% ==========================================
    %% VALIDADOES E ESQUEMAS (MODELS)
    %% ==========================================
    class UserModel { <<interface>> + schema: JoiSchema }
    class UserSensitiveModel { <<interface>> + schema: JoiSchema }
    class RunningBeltModel { <<interface>> + schema: JoiSchema }
    class RunRecordModel { <<interface>> + schema: JoiSchema }
    class SyncQueueModel { <<interface>> + schema: JoiSchema }
    class AuditModel { <<interface>> + schema: JoiSchema }
    class StatisticsModel { <<interface>> + schema: JoiSchema }
    class ReportModel { <<interface>> + schema: JoiSchema }

    %% ==========================================
    %% RELACIONAMENTOS DE DEPENDÊNCIA
    %% ==========================================
    RegistroController ..> RegistroService : usa
    PainelController ..> EstatisticaService : usa
    RelatorioController ..> RelatorioService : usa
    UsuarioController ..> UsuarioService : usa

    UsuarioService ..> UserRepository : usa
    UsuarioService ..> UserSensitiveRepository : usa

    RegistroService ..> RunRecordRepository : usa
    RegistroService ..> RunningBeltRepository : usa
    RegistroService ..> AuditRepository : usa
    RegistroService ..> SyncQueueRepository : usa
    RegistroService ..> StatisticsRepository : atualiza dados acumulados
    RegistroService ..> RunRecordModel : valida com

    EstatisticaService ..> StatisticsRepository : consulta dados agregados
    EstatisticaService ..> TeamRepository : usa
    EstatisticaService ..> UserRepository : usa
    EstatisticaService ..> StatisticsModel : valida com

    RelatorioService ..> ReportRepository : usa
    RelatorioService ..> StatisticsRepository : extrai dados
    RelatorioService ..> ReportModel : valida com

    %% Vinculação de modelos restantes para completar a cobertura do DER
    UsuarioService ..> UserModel : valida com
    UsuarioService ..> UserSensitiveModel : valida com
    RegistroService ..> RunningBeltModel : valida com
    RegistroService ..> SyncQueueModel : valida com
    RegistroService ..> AuditModel : valida com