# Task Manager

Gerenciador de Tarefas (Task Manager) com Next.js 14 e PostgreSQL.

Esta branch contém a aplicação, o Dockerfile e os manifests Kubernetes usados
para executá-la localmente.

## Demonstração

![Task Manager UI](https://via.placeholder.com/800x500?text=Task+Manager+UI)

## Funcionalidades

- **CRUD completo** de tarefas (Create, Read, Update, Delete)
- **Status das tarefas**: pendente, em-andamento, concluída
- **Prioridades**: baixa, média, alta
- **Dashboard**: estatísticas e filtros
- **API REST**: endpoints para integração
- **Health check**: endpoint para monitoramento

## Stack Tecnológica

- **Frontend**: Next.js 14 (App Router), Tailwind CSS
- **Backend**: Next.js API Routes
- **Banco de dados**: PostgreSQL 15
- **Containerização**: Docker, Docker Compose
- **Testes**: Jest

## Estrutura do Projeto

```
task-manager/
├── app/              # App Next.js (App Router)
│   ├── api/          # API Routes
│   ├── layout.js     # Layout raiz
│   ├── page.js       # Página principal (UI)
│   └── globals.css   # Estilos globais Tailwind
├── lib/              # Funções de banco de dados
│   ├── db.js         # Pool PostgreSQL
│   └── dbConfig.js   # Configuração de conexão
├── tests/            # Testes Jest para API
├── Dockerfile        # Build da imagem Docker
├── docker-compose.yml # Ambiente local com Docker Compose
├── next.config.js    # Config Next.js
├── tailwind.config.js # Config Tailwind CSS
└── package.json      # Dependências
```

## Execução da Aplicação

Execute os comandos a seguir a partir da pasta `task-manager/`. A aplicação
usa PostgreSQL para armazenar as tarefas e fica disponível na porta `3000`.

### Pré-requisitos

- Docker e Docker Compose
- Node.js 20+ (opcional, para desenvolvimento local sem Docker)

### Com Docker Compose (Recomendado)

```bash
# Iniciar containers
docker compose up -d

# Acessar a aplicação
open http://localhost:3000

# Verificar health check
curl http://localhost:3000/api/health

# Se precisar resetar o banco de dados
docker compose down -v
docker compose up -d
```

### Desenvolvimento Local

Instale as dependências e defina as variáveis de conexão com o PostgreSQL. O
banco precisa estar acessível em `localhost:5432`.

```bash
# Instalar dependências
npm install

# Configurar a conexão com o PostgreSQL
cat > .env.local <<'EOF'
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_NAME=task_manager
DATABASE_USER=admin
DATABASE_PASSWORD=admin
EOF

# Iniciar o servidor de desenvolvimento
npm run dev
```

Acesse `http://localhost:3000`. Para interromper o servidor, pressione
`Ctrl+C`.

## API REST

### Endpoints

| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/api/tasks` | Listar todas as tarefas |
| GET | `/api/tasks/:id` | Buscar tarefa por ID |
| POST | `/api/tasks` | Criar nova tarefa |
| PUT | `/api/tasks/:id` | Atualizar tarefa |
| DELETE | `/api/tasks/:id` | Deletar tarefa |
| GET | `/api/health` | Health check |

### Exemplo de Request

```bash
# Criar tarefa
curl -X POST http://localhost:3000/api/tasks \
  -H "Content-Type: application/json" \
  -d '{"title":"Minha tarefa","description":"Descrição","priority":"alta","status":"pendente"}'

# Listar tarefas
curl http://localhost:3000/api/tasks
```

## Testes

```bash
# Antes de rodar testes, iniciar o servidor
npm run dev &

# Rodar testes
npm test
```

> Nota: Os testes precisam que o servidor esteja rodando em `http://localhost:3000`

## Docker

### Build da Imagem

```bash
docker build -t task-manager .
```

### Executar com Docker

```bash
docker run -p 3000:3000 task-manager
```

Esse comando executa apenas a aplicação. Para iniciar a aplicação e o
PostgreSQL juntos, use a opção Docker Compose acima.

## Deploy em Kubernetes (k3d)

Os manifests em `k8s/` executam a aplicação com três réplicas, um PostgreSQL
interno, ConfigMap, Secret e Service NodePort.

```bash
# Criar o cluster, se necessário
k3d cluster create task-manager --servers 1 --agents 1 -p "8080:30080@loadbalancer"

# Construir e importar a imagem local
docker build -t task-manager:local .
k3d image import task-manager:local -c task-manager

# Aplicar os recursos e aguardar os Deployments
kubectl apply -f k8s/configmap.yaml -f k8s/secret.yaml -f k8s/postgres.yaml -f k8s/task-manager.yaml
kubectl rollout status deployment/postgres
kubectl rollout status deployment/task-manager
kubectl get pods,svc,deployments

# Verificar a aplicação
curl http://localhost:8080/api/health

# Escalar para quatro réplicas e verificar o resultado
kubectl scale deployment/task-manager --replicas=4
kubectl get deployment task-manager
kubectl get pods -l app=task-manager

# Coletar logs de um Pod e o describe do Deployment
POD=$(kubectl get pods -l app=task-manager -o jsonpath='{.items[0].metadata.name}')
kubectl logs "$POD"
kubectl describe deployment task-manager
```

## Licença

MIT
