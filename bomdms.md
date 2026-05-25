```mermaid
classDiagram

    class User {
        - id: number
        - name: string
        - email: string
        - role: string
    }

    class Team {
        - id: number
        - name: string
        - color: string
        + getTotalKm() number
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
        - startTime: Date
        - endTime: Date
        - isFinished: boolean
    }

    class Checkpoint {
        - id: number
        - runRecordId: number
        - currentKm: number
        - registeredAt: Date
    }

    Team "1" *-- "2" RunningBelt
    RunningBelt "1" o-- "0..*" RunRecord
    User "1" o-- "0..*" RunRecord
    RunRecord "1" *-- "1..*" Checkpoint

    class UserRepository {
        + findById(id) User
        + save(user) User
    }

    class TeamRepository {
        + findById(id) Team
        + findAll() Team[]
    }

    class RunningBeltRepository {
        + findById(id) RunningBelt
    }

    class RunRecordRepository {
        + findActiveRecord(beltId) RunRecord
        + save(record) RunRecord
    }

    class CheckpointRepository {
        + save(checkpoint) Checkpoint
        + findByRunRecord(runRecordId) Checkpoint[]
    }

    class RegistroService {
        - userRepo: UserRepository
        - beltRepo: RunningBeltRepository
        - runRecordRepo: RunRecordRepository
        - checkpointRepo: CheckpointRepository
        + iniciarTurno(userId, beltId) RunRecord
        + registrarCheckpoint(runRecordId, km) Checkpoint
        + encerrarTurno(runRecordId, kmFinal) RunRecord
    }

    class EstatisticaService {
        - teamRepo: TeamRepository
        - runRecordRepo: RunRecordRepository
        - checkpointRepo: CheckpointRepository
        + calcularPlacarEquipes() object
    }

    RegistroService ..> UserRepository
    RegistroService ..> RunningBeltRepository
    RegistroService ..> RunRecordRepository
    RegistroService ..> CheckpointRepository
    
    EstatisticaService ..> TeamRepository
    EstatisticaService ..> RunRecordRepository
    EstatisticaService ..> CheckpointRepository

    class RegistroController {
        - registroService: RegistroService
        + POST /turnos/iniciar iniciar()
        + POST /turnos/:id/checkpoints checkpoint()
        + POST /turnos/:id/encerrar encerrar()
    }

    class PainelController {
        - estatisticaService: EstatisticaService
        + GET /placar obterPlacar()
    }

    RegistroController --> RegistroService
    PainelController --> EstatisticaService