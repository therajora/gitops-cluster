# Applications Management

Este diretório contém as aplicações gerenciadas pelo ArgoCD usando o padrão App of Apps.

## Estrutura

```
apps/
├── app-of-apps.yaml          # Aplicação principal que gerencia todas as outras
└── applications/             # Aplicações individuais
    ├── nginx-ingress-app.yaml
    ├── repository-config.yaml
    └── example-app.yaml      # Exemplo de aplicação
```

## Como Funciona

1. **App of Apps**: O arquivo `app-of-apps.yaml` define uma aplicação ArgoCD que monitora o diretório `applications/`
2. **Aplicações Individuais**: Cada arquivo YAML em `applications/` define uma aplicação específica
3. **Sincronização Automática**: Quando você adiciona/remove arquivos em `applications/`, o ArgoCD sincroniza automaticamente

## Adicionando Novas Aplicações

### 1. Criar Arquivo de Aplicação

```yaml
# apps/applications/minha-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: minha-app
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://charts.example.com
    targetRevision: 1.0.0
    chart: minha-app
    helm:
      values: |
        # configurações aqui
  destination:
    server: https://kubernetes.default.svc
    namespace: minha-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

### 2. Commit e Push

```bash
git add apps/applications/minha-app.yaml
git commit -m "Add minha-app application"
git push origin main
```

### 3. Verificar no ArgoCD

A aplicação aparecerá automaticamente na interface do ArgoCD e será sincronizada.

## Tipos de Fonte Suportados

### Helm Charts
```yaml
source:
  repoURL: https://charts.example.com
  chart: chart-name
  targetRevision: 1.0.0
  helm:
    values: |
      key: value
```

### Kustomize
```yaml
source:
  repoURL: https://github.com/user/repo
  targetRevision: main
  path: kustomize/path
```

### Diretório Git
```yaml
source:
  repoURL: https://github.com/user/repo
  targetRevision: main
  path: manifests/
```

## Configurações Importantes

- **finalizers**: Sempre incluir para permitir limpeza adequada
- **syncPolicy.automated**: Habilita sincronização automática
- **syncOptions.CreateNamespace**: Cria namespace automaticamente
- **retry**: Configuração de retry para falhas temporárias

## Exemplos de Aplicações

### Monitoramento
- Prometheus + Grafana
- Jaeger (tracing)
- ELK Stack

### Bancos de Dados
- PostgreSQL
- Redis
- MongoDB

### Aplicações Web
- Nginx
- Apache
- Node.js apps

### APIs
- REST APIs
- GraphQL
- Microserviços
