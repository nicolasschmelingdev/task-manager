# Jenkins - Guia de Configuração e Pipeline

Este guia descreve como subir o Jenkins com Docker Compose e executar o pipeline definido em `Jenkinsfile` na raiz do repositório.

## Pré-requisitos
- Docker e Docker Compose instalados
- Porta 8080 livre (Jenkins), 50000 (JNLP) opcional
- Repositório com:
  - `Jenkinsfile`
  - `docker-compose.yml`
  - Backend: `task-manager-backend/`
  - Frontend: `task-manager-frontend/`

## Subindo o Jenkins
1. Suba os serviços (inclui Jenkins):
   ```bash
   docker compose up -d
   ```
2. Acesse: http://localhost:8080
3. Pegue a senha inicial (admin):
   - Pela UI do container (Logs) ou via CLI:
     ```bash
     docker logs jenkins 2>&1 | grep -A2 "Please use the following password"
     ```
4. Instale os plugins sugeridos + adicione “Docker Pipeline” (recomendado).

## Permitir que o Jenkins use o Docker do host
Para que as etapas “Docker Build” e “Deploy” do `Jenkinsfile` funcionem, o Jenkins precisa acessar o Docker do host.

- Verifique se o serviço `jenkins` em `docker-compose.yml` monta o socket do Docker:
  ```yaml
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    restart: unless-stopped
    user: "0:0"
    environment:
      JAVA_OPTS: "-Djava.util.logging.config.file=/var/jenkins_home/log.properties"
    ports:
      - "8080:8080"
      - "50000:50000"
    volumes:
      - jenkins-home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock   # <--- adicione esta linha para builds/deploy via Docker
  ```
Após alterar o compose, recrie o serviço Jenkins:
```bash
docker compose up -d --force-recreate jenkins
```

## Criando o Pipeline no Jenkins
1. No Jenkins, clique em “New Item” > “Pipeline”.
2. Dê um nome (por ex. `task-manager-pipeline`) e selecione “Pipeline”.
3. Em “Build Triggers”, habilite “GitHub hook trigger” se for usar webhooks (opcional).
4. Em “Pipeline” > “Definition”: “Pipeline script from SCM”.
5. Selecione “Git” e informe a URL do repositório. Se privado, configure credenciais.
6. Em “Script Path”, informe `Jenkinsfile` (padrão na raiz do repo).
7. Salve e clique em “Build Now”.

## O que o Jenkinsfile faz
Arquivo: `Jenkinsfile`
- Parâmetros:
  - `DOCKER_BUILD` (default: true): constrói a imagem Docker do backend
  - `DEPLOY` (default: false): executa `docker compose up -d --build`
- Estágios:
  1. **Checkout**: baixa o código do repositório
  2. **Backend - Build & Test**: roda `mvn clean verify` em `maven:3.9-eclipse-temurin-21`
  3. **Frontend - Build**: roda `npm ci && npm run build` em `node:20-alpine`
  4. **Docker Build** (opcional): `docker build` da imagem do backend
  5. **Deploy (docker compose)** (opcional): `docker compose up -d --build`
- Artefatos: publica `task-manager-backend/target/*.jar` ao final

## Comandos úteis
- Ver logs do Jenkins:
  ```bash
  docker logs -f jenkins
  ```
- Reiniciar somente o Jenkins:
  ```bash
  docker compose restart jenkins
  ```
- Derrubar o stack:
  ```bash
  docker compose down
  ```

## Otimizações (opcionais)
- `task-manager-backend/Dockerfile` já está com `-Dmaven.test.skip=true` para pular compilação de testes em build de imagem. Para ainda mais performance, podemos usar cache do Maven com BuildKit.
- No `Jenkinsfile`, troque `mvn clean verify` por `mvn -DskipTests clean package` se quiser acelerar builds no CI.
- Testes de integração com Testcontainers (Oracle) podem ser isolados por profiles (ex.: `-Punit` / `-Pintegration`).

## Troubleshooting
- **Porta 8080 ocupada**: pare outro serviço que use 8080 ou mude a porta do Jenkins no `docker-compose.yml`.
- **Permissão para usar Docker**: confirme se o socket `/var/run/docker.sock` está montado no container Jenkins e o usuário está como `root` (já definido em `docker-compose.yml`).
- **Falhas no Maven (rede)**: o primeiro build pode demorar por download de dependências; repetições serão mais rápidas pelo cache.

---
Em caso de dúvidas ou para automatizar as otimizações acima, me avise que eu aplico as alterações no repositório.
