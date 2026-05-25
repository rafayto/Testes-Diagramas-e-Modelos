```classDiagram

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
    RunningBelt --> StatusKind
    RunningBelt "2" --> "1" Team
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
        + findByTe