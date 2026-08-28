# Evidências do deploy Kubernetes

Aplicação: `task-manager`
Cluster: `k3d-task-manager`
Data da execução: 28/08/2026

## 1. Deployment inicial

Manifesto utilizado: [task-manager.yaml](../k8s/task-manager.yaml)

Comando:

```bash
kubectl apply -f k8s/configmap.yaml -f k8s/secret.yaml -f k8s/postgres.yaml -f k8s/task-manager.yaml
kubectl rollout status deployment/task-manager
```

Resultado:

```text
deployment.apps/task-manager created
service/task-manager created
deployment "task-manager" successfully rolled out
```

O Deployment foi criado inicialmente com 3 réplicas, conforme `spec.replicas: 3`.

## 2. Service NodePort

Manifesto utilizado: [task-manager.yaml](../k8s/task-manager.yaml)

Resultado coletado:

```text
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)
service/task-manager   NodePort   10.43.191.38   <none>   3000:30080/TCP
```

A aplicação ficou acessível em `http://localhost:8080`, usando o mapeamento do k3d `8080:30080@loadbalancer`.

## 3. ConfigMap injetado

Manifesto: [configmap.yaml](../k8s/configmap.yaml)

O Deployment referencia o ConfigMap com `envFrom.configMapRef`.

Verificação feita dentro de um Pod:

```text
APP_ENV=kubernetes
DATABASE_HOST=postgres
DATABASE_PORT=5432
DATABASE_NAME=task_manager
```

O PostgreSQL e as credenciais usadas pela aplicação estão definidos em [postgres.yaml](../k8s/postgres.yaml) e [secret.yaml](../k8s/secret.yaml).

## 4. Escala para 4 réplicas

Comandos:

```bash
kubectl scale deployment/task-manager --replicas=4
kubectl get deployment task-manager
kubectl get pods -l app=task-manager
```

Resultado:

```text
deployment.apps/task-manager scaled
deployment "task-manager" successfully rolled out
NAME           READY   UP-TO-DATE   AVAILABLE
 task-manager  4/4     4            4
```

Os quatro Pods ficaram com status `Running` e zero reinicializações.

## 5. Logs e describe

Pod usado: `task-manager-786c8959cd-4pzqm`

Comandos:

```bash
POD=$(kubectl get pods -l app=task-manager -o jsonpath='{.items[0].metadata.name}')
kubectl logs "$POD"
kubectl describe deployment task-manager
```

Trecho dos logs:

```text
> Ready on http://0.0.0.0:3000
Database initialization attempt 1/15 failed, retrying in 2s...
Database initialized successfully
```

Resumo do `describe`:

```text
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
Image:                  task-manager:local
Port:                   3000/TCP
Environment Variables from:
  task-manager-config  ConfigMap
  postgres-secret      Secret
Liveness:               http-get http://:3000/api/health
Readiness:              http-get http://:3000/api/health
Conditions:
  Available            True
  Progressing          True
```

## Verificação da aplicação

```text
GET /                 HTTP 200
GET /api/health       HTTP 200
GET /api/tasks        HTTP 200
/api/health           {"status":"ok","service":"task-manager"}
/api/tasks            {"tasks":[]}
```

## Print da tela

Para capturar o print exigido, abra `http://localhost:8080` no navegador enquanto o cluster estiver ativo e salve a imagem nesta pasta, por exemplo como `tela-aplicacao.png`. O ambiente de execução não possui Chromium/Chrome instalado para gerar o PNG automaticamente.
