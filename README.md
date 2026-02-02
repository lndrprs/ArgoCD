<div align="Center"> 
<br>

<h4>

░█████╗░██████╗░░██████╗░░█████╗░░█████╗░██████╗░
██╔══██╗██╔══██╗██╔════╝░██╔══██╗██╔══██╗██╔══██╗
███████║██████╔╝██║░░██╗░██║░░██║██║░░╚═╝██║░░██║
██╔══██║██╔══██╗██║░░╚██╗██║░░██║██║░░██╗██║░░██║
██║░░██║██║░░██║╚██████╔╝╚█████╔╝╚█████╔╝██████╔╝
╚═╝░░╚═╝╚═╝░░╚═╝░╚═════╝░░╚════╝░░╚════╝░╚═════╝░
</div>

----

<details>
  <summary><b> 1. Tópicos </b></summary>
<div align="Left">  
<br>  

A1.1 - Conceitos de GitOps 
> - GitOps é um modelo operacional para  gerenciar infraestrutura e aplicações kubernetes, onde:
>   - Git é a única fonte de verdade (Single Source of Truth);
>   - Todo o estado desejado do sistema é descrito declarativamente (YAML, Helm, Kustomize);
>   - Um agente automático no cluster garante que o estado real convirja para o estado descrito no Git.
> - No GitOps, o cluster nunca é alterado manualmente de forma permanente. 
>
> - Existe a questão de Estado Desejado vs Estado Atual:
>   - Estado Desejado: O que está versionado no Git;
>   - Estado Atual: O que está rodando no Cluster. 
> - O ArgoCD lê o estado desejado no Git, compara com o estado atual do cluster, e executa a correção (Sync), quando há divergência;
> - Esse processo de sincronização, é a "reconciliação contínua". 
>
> - GitOps usa o modelo Pull:
>   - ArgoCD PUXA as mudanças do Git;
>   - O Git não acessa o Cluster.
> - Benefícios:
>   - Cluster não precisa expor credenciais;
>   - Maior segurança;
>   - Melhor controle de auditoria. 

A1.2 - Argo CD
> - Controlador GitOps declarativo para Kubernetes:
>   - Monitora repositórios Git;
>   - Compara Manifests com o Cluster;
>   - Aplica mudanças automaticamente ou sob aprovação;
>   - Mantém o cluster sincronizado com o Git. 
> - O Argo CD não substitui o CI - Continuous Integration. 
>
> - Casos de Uso:
>   - Provisionamento de Aplicações;
>   - Gestão de Infraestrutura Kubernetes;
>   - Padronização de Ambientes;
>   - Operação Multi-Cluster;
>   - Governança Corporativa. 
>
> - Ecossistema Argo
>   - Argo CD: Provisionamento GitOps;
>   - Argo Workflows: Pipelines e Jobs;
>   - Argo Rollouts: Deploys Progressivos (Canary, Blue/Green);
>   - Argo Events: Event-Driven Workflows.

A1.3 - Arquitetura do Argo CD
> - Git Repo -> Servidor do Repositório -> Controlador de Aplicação | Servidor API Kubernetes -> Cluster Alvo. 
>
> - API Server (argocd-server)
>   - Interface central do Argo CD;
>   - Fornece UI Web, API Rest /gRPC, e Autenticação / RBAC;
>   - Não Aplica recursos diretamente no Cluster. 
>
> - Servidor do Repositório (argocd-repo-server)
>   - Acessa repositórios Git / Helm / OCI;
>   - Renderiza Manifestos: Helm templates, Kustomize Builds, Jsonnet;
>   - Entre manifestos prontos ao Controlador. 
> - Obs.: Não conhece o cluster, apenas gera o YAML. 
>
> - Controlador de Aplicação (argocd-application-controller)
>   - Observa CRDs (Custom Resource Definition) Application;
>     - Compara Manifestos renderizados, e recursos reais no cluster. 
>   - Executa: Sincronização, Self-Healing e Prune.
>
> - Redis
>   - Cache de Estado;
>   - Reduz chamadas ao Kubernetes API;
>   - Melhora performance e escalabilidade. 
>
> - Interação com o Kubernetes
>   - O ArgoCD: 
>     - Cria e gerencia recursos Kubernetes;
>     - Usa Service Accounts;
>     - Respeita RBAC do Cluster. 
>   - Toda ação passas pelo Kubernetes API Server. 
>
> - Características Arquiteturais Importantes
>   - Totalmente declarativo;
>   - Stateless (Exceto Cache);
>   - Altamente Escalável;
>   - Seguro por Design (Modelo Pull).

A1.4 - Application 
> - Application é uma CRD (Custom Resource Definition), do Argo CD;
>   - Se trata de uma aplicação declarativa que deve existir em um cluster Kubernetes, conforme descrito no Git;
>   - Cada Aplicação define:
>     - De onde vem o código: Git, Helm, etc.;
>     - Para onde será aplicado: Cluster + Namespace;
>     - Como será sincronizado. 
>
> - A Aplicação funciona como um "contrato" GitOps:
>   - Git diz o que deve existir;
>   - Argo CD garante que aquilo exista. 
> - Caso ocorra alguma alteração manual no cluster:
>   - O Argo CD detecta;
>   - Marca como "OutOfSync";
>   - Pode corrigir automaticamente - Self-Heal. 
>
> - Estrutura Conceitual de uma Aplicação
>   - Source: Repositório, Caminho, Tipo (Helm, Kustomize, YAML);
>   - Destination: Cluster, Namespace;
>   - Sync Policy: Manual ou Automática;
>   - Status: Sync Status, Health Status. 
>
> - Granularidade
>   - Uma Aplicação não precisa ser um único microserviço;
>   - Pode representar:
>     - Um serviço;
>     - Conjunto de Serviços;
>     - Infraestrutura (Ingress, CRDs, Operators).
> - Boa Prática: Uma aplicação por domínio lógico. 

A1.5 - Project (AppProject)
> - AppProject é uma CRD usada para Organização, Isolamento e Segurança.
> - Define:
>   - O que uma Aplicação pode fazer;
>   - Onde ela pode rodar;
>   - De onde ela pode ler. 
>
> - Limites de Segurança
>   - Um Projeto controla:
>     - Repositórios Permitidos;
>     - Clusters Permitidos;
>     - Namespaces Permitidos;
>     - Tipos de Recursos Kubernetes Permitidos. 
>
> - Uso típico de AppProject
>   - Separação por:
>     - Times;
>     - Ambientes;
>     - Domínios de Negócio. 
>    - Multi-Tenant;
>    - Ambientes Regulados. 

A1.6 - Repositório 
> - Repositório no ArgoCD é uma fonte de manifestos declarativos. 
>   - Pode ser:
>     - Git (HTTPS ou SSH);
>     - Helm;
>     - OCI Registry (Charts ou Manifests).
> - O Argo CD não altera o repositório, apenas lê e observa mudanças. 
>
> - Segurança do Repositório
>   - Credenciais ficam no Argo CD;
>   - Git nunca acessa o Cluster;
>   - Permite auditoria e Controle. 

A1.7 - Destination (Cluster e Namespace)
> - Cluster
>   - Pode ser onde o Argo CD está instalado, ou Cluster Externos;
>   - Cada Cluster é registrado explicitamente. 
>
> - Namespace
>   - Parte obrigatório do destino;
>   - Ajuda a isolar aplicações;
>   - Governança forte em ambientes corporativos. 

A1.8 - Manifestos Suportados 
> - ArgoCD é agnóstico à ferramenta de template, desde que o resultado final seja YAML Kubernetes válido. 
> - Tipos:
>   - YAML Puro;
>   - Helm;
>   - Kustomize;
>   - Jsonnet;
>   - Directory-based Apps;

A1.9 - Instalação e Acesso
> - O ArgoCD sempre é instalado dentro de um cluster Kubernetes; 
> - Normalmente em um namespace dedicado: argocd
> - Pode gerenciar o próprio cluster, ou clusters remotos. 
>
> - Modelos de Instalação
>   - Manifestos Oficiais: YAMLs prontos;
>   - Helm Chart;
>   - Operator (Especialmente no OpenShift).
>
> - Acesso
>   - Através da UI Web;
>   - CLI (argocd);
>   - API (REST / gRPC).
>
> - Auteneticação
>   - Usuários Locais;
>   - Tokens;
>   - Integração com Identity Providers. 

</div>
</details>

----