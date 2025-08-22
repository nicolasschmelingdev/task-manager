# Task Manager – Desafio Técnico (Full Stack)

Aplicação web para gerenciamento de tarefas. Atende aos requisitos do teste prático: CRUD completo de tarefas, listagem paginada e ordenada, integração com Oracle, CI com Jenkins e entrega via Docker Compose.

## Visão Geral do Projeto
- __Frontend__: `task-manager-frontend/` (Angular 20, CSS). Servido em dev pelo `ng serve` (porta 4200).
- __Backend__: `task-manager-backend/` (Spring Boot 3, REST). Expõe a API em `8081` sob o prefixo `/api`.
- __Banco__: Oracle XE (imagem `gvenzl/oracle-free`).
- __DevOps__: Docker Compose para orquestração e Jenkins para CI com `Jenkinsfile` na raiz.

## Como executar (Docker Compose)
Pré-requisitos: Docker e Docker Compose. Na raiz do repositório:
```bash
docker compose up -d --build
```
URLs de acesso:
- Frontend: http://localhost:4200
- API: http://localhost:8081/api
- Jenkins: http://localhost:8080
- Oracle XE: localhost:1521 (service `XEPDB1`)

Parar/remover serviços:
```bash
docker compose down
```

## Como executar localmente (sem Docker)
Backend:
```bash
cd task-manager-backend
./mvnw clean package
java -jar target/task-manager-backend-*.jar
```
Frontend:
```bash
cd task-manager-frontend
npm ci
npm start
```
O frontend usa a base `http://localhost:8081/api` (arquivo `task-manager-frontend/src/environments/environment.ts`).

## Requisitos do Sistema (atendidos)
- __CRUD de Tarefas__: criar, listar, detalhar, atualizar e excluir tarefas.
  - Campos: `id`, `titulo`, `descricao`, `status` (`PENDENTE`, `EM_ANDAMENTO`, `CONCLUIDA`), `dataCriacao`, `dataAtualizacao`, `atualizadoPor`.
- __Listagem Paginada e Ordenada__: parâmetros `page`, `size`, `sort` (ex.: `sort=dataCriacao,desc` ou `sort=status,asc`).
- __Banco Relacional Oracle + Hibernate/JPA__: mapeamentos JPA, schema gerenciado por Flyway.

## Exemplos de uso da API (HTTP)
Base URL: `http://localhost:8081/api`

- __Criar tarefa__
  - POST `/tasks`
  - Body JSON (ex.):
    ```json
    {"titulo":"Estudar Spring","descricao":"Ler docs","status":"PENDENTE"}
    ```

- __Listar tarefas (paginado/ordenado)__
  - GET `/tasks?page=0&size=10&sort=dataCriacao,desc`

- __Buscar por id__
  - GET `/tasks/{id}`

- __Atualizar tarefa__
  - PUT `/tasks/{id}`
  - Body JSON (ex.):
    ```json
    {"titulo":"Estudar Spring","descricao":"Revisar JPA","status":"EM_ANDAMENTO"}
    ```

- __Excluir tarefa__
  - DELETE `/tasks/{id}`

Obs.: Endpoints seguem convenções REST; o backend expõe as rotas sob `/api`. Caso necessário, consulte o código em `task-manager-backend/src/main/`.

## Decisões Técnicas
- __Angular 20__: versão atual, CLI para scaffolding e Material opcional.
- __Spring Boot 3 + Java 21__: performance e recursos modernos.
- __Hibernate/JPA__: abstração de persistência e validação com Bean Validation.
- __Flyway__: versionamento de schema e migrações confiáveis para Oracle.
- __JWT (io.jsonwebtoken)__: autenticação stateless; variável `JWT_SECRET` configurável.
- __Docker Compose__: rede única para comunicação entre serviços (backend⇄Oracle, frontend→backend).
- __Jenkinsfile__: pipeline declarativo versionado junto ao código.

## Performance
- __Maven cache em Docker__ (opcional): pode-se habilitar `--mount=type=cache` para acelerar builds.
- __HikariCP__: pool de conexões configurável via propriedades Spring (`HIKARI_CONNECTION_TIMEOUT`, etc.).
- __Compressão HTTP & GZIP__ (via reverse proxy em produção) recomendada.
- __Paginação padrão__: evita respostas grandes por padrão (`size` configurável).

## Segurança
- __JWT__: token assinado e com expiração (`JWT_EXPIRATION_MILLIS`).
- __CORS__: habilitar para o origin do frontend (`http://localhost:4200`).
- __Validação de entrada__: Bean Validation na camada REST.
- __Perfis__: `application-{profile}.properties` para separar dev/produção.
- __Headers HTTP__ (produção): recomendar proxy (Nginx) para HSTS, X-Frame-Options, etc.

## Jenkins (CI)
- Serviço disponível em `docker-compose.yml` (porta 8080).
- Pipeline em `Jenkinsfile` com estágios:
  - Checkout do repositório
  - Build/Test do backend (Maven)
  - Build do frontend (Node)
  - Build de imagem Docker do backend (opcional)
  - Deploy via `docker compose` (opcional)
- Guia detalhado: `JENKINS.md` (inclui como obter a senha inicial e como permitir acesso ao Docker do host).

## Arquitetura & Microsserviços
- O backend está containerizado e desacoplado do frontend; comunicação via HTTP sobre rede Docker.
- A estrutura é preparada para evolução para microsserviços (ex.: extrair auth/usuarios, tarefas, notificações) mantendo contratos REST e versionamento de schema com Flyway.

## Metodologias Ágeis
- O projeto foi pensado para fluxo Scrum/Kanban:
  - Backlog de features (CRUD, paginação, ordenação, autenticação, CI/CD).
  - Divisão em sprints curtos com entregas incrementais.
  - Kanban para acompanhar status (To Do / Doing / Done) e limites de WIP.

## Estrutura do Repositório
```
./docker-compose.yml
./Jenkinsfile
./JENKINS.md
./task-manager-backend/
  ├─ Dockerfile
  └─ src/
./task-manager-frontend/
  ├─ package.json
  └─ src/
```

## Troubleshooting
- Primeiro build do backend em Docker pode demorar (download de dependências).
- Se o Jenkins não mostrar a senha inicial, veja `JENKINS.md` para alternativas.
- Conflitos de porta: ajuste mapeamentos em `docker-compose.yml`.
- Oracle pode levar alguns segundos para ficar “healthy”; o backend aguarda via `depends_on`.

---
Para dúvidas, melhorias (CORS no backend, imagem de produção do frontend com Nginx, otimizações de build) ou adequações específicas de ambiente, abra uma issue ou entre em contato.
