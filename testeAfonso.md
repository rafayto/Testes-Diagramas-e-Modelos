@startuml DiagramaClassesArquitetural
' ============================================================
'  Subseção 3.2.3.1 — Diagrama de Classes Arquitetural
'  Componentes centrais das camadas:
'   Controller • Service • Repository • Model
'  Domínio: Sistema de Empréstimo de Livros (Biblioteca)
'  Stack: TypeScript (NestJS/Express + TypeORM-like)
' ============================================================

title Diagrama de Classes Arquitetural — Biblioteca (TypeScript)

skinparam shadowing false
skinparam roundCorner 8
skinparam classAttributeIconSize 0
skinparam packageStyle rectangle
skinparam class {
  BackgroundColor #FFFFFF
  BorderColor #333333
  ArrowColor #555555
}

' ====================== MODEL ======================
package "model" <<Folder>> #F3E8FF {

  class Usuario {
    - id: string
    - nome: string
    - email: string
    - tipo: TipoUsuario
    + podePegarEmprestado(): boolean
  }

  class Livro {
    - id: string
    - titulo: string
    - autor: string
    - isbn: string
    - exemplaresDisponiveis: number
    + estaDisponivel(): boolean
    + reservarExemplar(): void
    + devolverExemplar(): void
  }

  class Emprestimo {
    - id: string
    - usuarioId: string
    - livroId: string
    - dataRetirada: Date
    - dataPrevistaDevolucao: Date
    - dataDevolucao?: Date
    - status: StatusEmprestimo
    + estaAtrasado(): boolean
    + calcularMulta(): number
    + registrarDevolucao(data: Date): void
  }

  enum TipoUsuario {
    ALUNO
    PROFESSOR
    VISITANTE
  }

  enum StatusEmprestimo {
    ATIVO
    DEVOLVIDO
    ATRASADO
  }
}

' ==================== REPOSITORY ====================
package "repository" <<Folder>> #E8F8E8 {

  interface IUsuarioRepository {
    + findById(id: string): Promise<Usuario | null>
    + findByEmail(email: string): Promise<Usuario | null>
    + save(usuario: Usuario): Promise<Usuario>
  }

  interface ILivroRepository {
    + findById(id: string): Promise<Livro | null>
    + findByIsbn(isbn: string): Promise<Livro | null>
    + findDisponiveis(): Promise<Livro[]>
    + save(livro: Livro): Promise<Livro>
  }

  interface IEmprestimoRepository {
    + findById(id: string): Promise<Emprestimo | null>
    + findAtivosPorUsuario(usuarioId: string): Promise<Emprestimo[]>
    + save(emp: Emprestimo): Promise<Emprestimo>
  }

  class UsuarioRepository implements IUsuarioRepository {
    - orm: DataSource
    + findById(id)
    + findByEmail(email)
    + save(usuario)
  }

  class LivroRepository implements ILivroRepository {
    - orm: DataSource
    + findById(id)
    + findByIsbn(isbn)
    + findDisponiveis()
    + save(livro)
  }

  class EmprestimoRepository implements IEmprestimoRepository {
    - orm: DataSource
    + findById(id)
    + findAtivosPorUsuario(usuarioId)
    + save(emp)
  }
}

' ===================== SERVICE =====================
package "service" <<Folder>> #FFF7E0 {

  class UsuarioService {
    - usuarioRepo: IUsuarioRepository
    + cadastrar(dto: CriarUsuarioDTO): Promise<Usuario>
    + buscarPorId(id: string): Promise<Usuario>
  }

  class LivroService {
    - livroRepo: ILivroRepository
    + cadastrar(dto: CriarLivroDTO): Promise<Livro>
    + listarDisponiveis(): Promise<Livro[]>
  }

  class EmprestimoService {
    - empRepo: IEmprestimoRepository
    - livroRepo: ILivroRepository
    - usuarioRepo: IUsuarioRepository
    + realizarEmprestimo(dto: NovoEmprestimoDTO): Promise<Emprestimo>
    + devolverLivro(emprestimoId: string): Promise<Emprestimo>
    + listarPorUsuario(usuarioId: string): Promise<Emprestimo[]>
  }
}

' ==================== CONTROLLER ===================
package "controller" <<Folder>> #E8F4FD {

  class UsuarioController {
    - usuarioService: UsuarioService
    + POST /usuarios     criar(req, res)
    + GET  /usuarios/:id obter(req, res)
  }

  class LivroController {
    - livroService: LivroService
    + POST /livros            criar(req, res)
    + GET  /livros/disponiveis listar(req, res)
  }

  class EmprestimoController {
    - emprestimoService: EmprestimoService
    + POST /emprestimos              criar(req, res)
    + POST /emprestimos/:id/devolver devolver(req, res)
    + GET  /usuarios/:id/emprestimos listar(req, res)
  }
}

' ==================== DTO (apoio) ==================
package "dto" <<Folder>> #FFFFFF {
  class CriarUsuarioDTO {
    + nome: string
    + email: string
    + tipo: TipoUsuario
  }
  class CriarLivroDTO {
    + titulo: string
    + autor: string
    + isbn: string
    + exemplares: number
  }
  class NovoEmprestimoDTO {
    + usuarioId: string
    + livroId: string
  }
}

' ================= RELACIONAMENTOS =================

' Controllers dependem de Services
UsuarioController --> UsuarioService
LivroController --> LivroService
EmprestimoController --> EmprestimoService

' Services dependem das INTERFACES de Repository (DIP)
UsuarioService ..> IUsuarioRepository
LivroService ..> ILivroRepository
EmprestimoService ..> IEmprestimoRepository
EmprestimoService ..> ILivroRepository
EmprestimoService ..> IUsuarioRepository

' Repositories operam sobre entidades de Model
UsuarioRepository ..> Usuario
LivroRepository ..> Livro
EmprestimoRepository ..> Emprestimo

' Associações de domínio
Emprestimo "0..*" --> "1" Usuario : pertence a
Emprestimo "0..*" --> "1" Livro   : refere-se a
Usuario --> TipoUsuario
Emprestimo --> StatusEmprestimo

' Controllers consomem DTOs
UsuarioController ..> CriarUsuarioDTO
LivroController ..> CriarLivroDTO
EmprestimoController ..> NovoEmprestimoDTO

legend right
  Convenções:
  ──▶ associação / dependência forte
  ┄┄▶ dependência (uso) / implementação
  Services dependem de **interfaces** de repositório
  → Inversão de Dependência (DIP).
endlegend

@enduml