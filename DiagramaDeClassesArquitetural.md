```mermaid
classDiagram

    class UsersController {
        <<Controller>>
        -usersService: UsersService
        +users(req: Request, res: Response) void
        +usersById(req: Request, res: Response) void
        +usersTeam(req: Request, res: Response) void
    }
    class TeamsController {
        <<Controller>>
        -teamsService: TeamsService
        +teams(req: Request, res: Response) void
        +teamsRunningBelts(req: Request, res: Response) void
        +teamsStatistics(req: Request, res: Response) void
    }
    class RunningBeltsController {
        <<Controller>>
        -runningBeltsService: RunningBeltsService
        +runningBelts(req: Request, res: Response) void
        +runningBeltsRunRecords(req: Request, res: Response) void
    }
    class RunRecordsController {
        <<Controller>>
        -runRecordsService: RunRecordsService
        +runRecordStart(req: Request, res: Response) void
        +runRecordCheckpoint(req: Request, res: Response) void
        +runRecordEnd(req: Request, res: Response) void
        +runRecordById(req: Request, res: Response) void
        +runRecordsHistory(req: Request, res: Response) void
    }
    class StatisticsController {
        <<Controller>>
        -statisticsService: StatisticsService
        +statisticsTeams(req: Request, res: Response) void
        +statisticsUsers(req: Request, res: Response) void
        +statisticsRanking(req: Request, res: Response) void
    }
    class AdminController {
        <<Controller>>
        -adminService: AdminService
        +adminExportReport(req: Request, res: Response) void
    }

    class UsersService {
        <<Service>>
        -usersRepository: UsersRepository
        +getUsers() Promise
        +getUserById(id: string) Promise
        +getUserTeam(id: string) Promise
    }
    class TeamsService {
        <<Service>>
        -teamsRepository: TeamsRepository
        +getTeams() Promise
        +getTeamRunningBelts(teamId: string) Promise
        +getTeamStatistics(teamId: string) Promise
    }
    class RunningBeltsService {
        <<Service>>
        -runningBeltsRepository: RunningBeltsRepository
        +getRunningBelts() Promise
        +getRunningBeltRunRecords(id: string) Promise
    }
    class RunRecordsService {
        <<Service>>
        -runRecordsRepository: RunRecordsRepository
        +startRunRecord(body: any) Promise
        +checkpointRunRecord(id: string, body: any) Promise
        +endRunRecord(id: string, km_final: number) Promise
        +getRunRecordById(id: string) Promise
        +getRunRecordsHistory() Promise
    }
    class StatisticsService {
        <<Service>>
        -statisticsRepository: StatisticsRepository
        +getTeamsStatistics() Promise
        +getUsersStatistics() Promise
        +getStatisticsRanking() Promise
    }
    class AdminService {
        <<Service>>
        -adminRepository: AdminRepository
        +exportReport() Promise
    }

    class UsersRepository {
        <<Repository>>
        -db: Database
        +getUsers() Promise
        +getUserById(id: string) Promise
        +getUserTeam(id: string) Promise
    }
    class TeamsRepository {
        <<Repository>>
        -db: Database
        +getTeams() Promise
        +getTeamRunningBelts(teamId: string) Promise
        +getTeamStatistics(teamId: string) Promise
    }
    class RunningBeltsRepository {
        <<Repository>>
        -db: Database
        +getRunningBelts() Promise
        +getRunningBeltRunRecords(id: string) Promise
    }
    class RunRecordsRepository {
        <<Repository>>
        -db: Database
        +startRunRecord(body: any) Promise
        +checkpointRunRecord(id: string, body: any) Promise
        +endRunRecord(id: string, km_final: number) Promise
        +getRunRecordById(id: string) Promise
        +getRunRecordsHistory() Promise
    }
    class StatisticsRepository {
        <<Repository>>
        -db: Database
        +getTeamsStatistics() Promise
        +getUsersStatistics() Promise
        +getStatisticsRanking() Promise
    }
    class AdminRepository {
        <<Repository>>
        -db: Database
        +exportReport() Promise
    }

    class User {
        <<Model>>
        +id: number
        +nickname: string
        +picture: Buffer
        +team_id: number
        +role_kind_id: number
        +created_at: Date
    }
    class Team {
        <<Model>>
        +id: number
        +name: string
        +color: string
        +captain_id: number
        +analist_id: number
    }
    class RunningBelt {
        <<Model>>
        +id: number
        +team_id: number
        +name: string
        +status_kind_id: number
    }
    class RunRecord {
        <<Model>>
        +id: number
        +running_belt_id: number
        +user_id: number
        +km_inicial: number
        +km_final: number
        +epoch_time: Date
        +is_open: boolean
        +synced: boolean
    }
    class Statistics {
        <<Model>>
        +id: number
        +user_id: number
        +team_id: number
        +stats_kind_id: number
        +value: number
        +last_sync: Date
    }
    class Report {
        <<Model>>
        +id: number
        +generated_by: number
        +format: string
        +encoding: string
        +created_at: Date
    }

    UsersController --> UsersService : usa
    TeamsController --> TeamsService : usa
    RunningBeltsController --> RunningBeltsService : usa
    RunRecordsController --> RunRecordsService : usa
    StatisticsController --> StatisticsService : usa
    AdminController --> AdminService : usa

    UsersService --> UsersRepository : usa
    TeamsService --> TeamsRepository : usa
    RunningBeltsService --> RunningBeltsRepository : usa
    RunRecordsService --> RunRecordsRepository : usa
    StatisticsService --> StatisticsRepository : usa
    AdminService --> AdminRepository : usa

    UsersRepository ..> User : acessa
    TeamsRepository ..> Team : acessa
    RunningBeltsRepository ..> RunningBelt : acessa
    RunRecordsRepository ..> RunRecord : acessa
    StatisticsRepository ..> Statistics : acessa
    AdminRepository ..> Report : acessa

    User --> Team : pertence a
    RunningBelt --> Team : pertence a
    RunRecord --> User : feito por
    RunRecord --> RunningBelt : usa
    Statistics --> User : referencia
    Statistics --> Team : referencia
    Report --> User : gerado por
```
