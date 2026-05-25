```mermaid
classDiagram
    direction TB

%% ==================== CONTROLLERS ====================
    class RegistroController {
        - registroService: RegistroService
        + iniciar(req, res) : void
        + atualizarKm(req, res) : void
        + syncOffline(req, res) : void
    }

    class PainelController {
        - estatisticaService: EstatisticaService
        + obterDados(req, res) : void
        + gerarRelatorio(req, res) : void
    }

%% ==================== SERVICES ====================
    class RegistroService {
        - runRecordRepository: RunRecordRepository
        - syncQueueRepository: SyncQueueRepository
        - userRepository: UserRepository
        + iniciarTurno(userId, beltId) : RunRecord
        + atualizarKm(runRecordId, kmAtual) : void
        + processarFilaOffline() : void
    }

    class EstatisticaService {
        - statisticRepository: StatisticRepository
        - reportRepository: ReportRepository
        + consolidarKms() : void
        + gerarRelatorioDiario(userId, formato) : Report
    }

%% ==================== REPOSITORIES ====================
    class UserRepository {
        - db: Database
        + findById(id) : User
        + findSensitiveData(userId) : UserSensitive
    }

    class RunRecordRepository {
        - db: Database
        + findActive(beltId) : RunRecord
        + save(record) : RunRecord
    }

    class SyncQueueRepository {
        - db: Database
        + findPending() : SyncQueue[]
        + markAsSent(id) : void
    }

    class StatisticRepository {
        - db: Database
        + updateTeamStats(teamId, value) : void
    }

    class ReportRepository {
        - db: Database
        + save(report) : void
    }

%% ==================== MODELS (Esquemas) ====================
    class RunRecordModel {
        <<interface>>
        + schema: JoiSchema
    }

    class UserModel {
        <<interface>>
        + schema: JoiSchema
    }

%% ==================== RELACIONAMENTOS ====================
    
    %% Controllers usam Services
    RegistroController ..> RegistroService : usa
    PainelController ..> EstatisticaService : usa

    %% Services usam Repositories
    RegistroService ..> UserRepository : usa
    RegistroService ..> RunRecordRepository : usa
    RegistroService ..> SyncQueueRepository : usa
    
    EstatisticaService ..> StatisticRepository : usa
    EstatisticaService ..> ReportRepository : usa

    %% Services validam com Models
    RegistroService ..> RunRecordModel : valida com
    RegistroService ..> UserModel : valida com