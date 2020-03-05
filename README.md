# kubernetes101

Workshop Kubernetes 101




## Trabalhando com Namespaces

Os Namespaces é um objeto do Kubernestes utilizado para organizar o Cluster. Podem ser encarados como um diretótio para os objetos da API do Kubernetes; E, são uma maneira de dividir os recursos do cluster entre vários usuários (via cota de recursos).

Os Namespaces fornecem um escopo para nomes. Os nomes dos recursos precisam ser exclusivos dentro de um Namespace, mas não entre Namespaces. Os Namespaces não podem ser aninhados um no outro e cada recurso Kubernetes pode estar apenas em um Namespace.

Os namespaces proveem um escopo de segurança onde se pode aplicar regras de Role-base acces control (RBAC).

Como um folder tradicional, se apagamos um Namespace todos os recursos nele alocados são perdidos, por isso devemos ter muita atenção!

### Listando Namespaces

Para obter uma listas dos Namespaces de um Cluster usamos:

```$ kubectl get namespaces```

Para ter uma resumo do Namespace usamos:

```$ kubectl get namespaces <nome>```

Para obter informações detalhadas de um Namespace usamos:

```$ kubectl describe namespaces <nome>```

### Criando Namespaces

Para criar um Namespace precisamos ter um descritivo (lembrando que tudo no Kubernetes são objetos de API que precisamos ser descritos)

Criamos então um novo YAML chamado \<meu-namespace>.yaml

``` 
apiVersion: v1
kind: Namespace
metadata:
  name: <meu-namespace>
```

E executar:

```$ kubectl create f ./<meu-namespace>.yaml```

Alternativamente:

```$ kubectl create namespace <meu-namespace>```

### Subdividindo o Cluster com Namespaces

Para este exercício, criaremos dois namespaces Kubernetes adicionais para armazenar nosso conteúdo.

O cenário em que uma organização está usando um cluster Kubernetes compartilhado para casos de uso de desenvolvimento e produção. Nós vamos criar dois novos namespaces para manter nosso trabalho.

Criando o Namespace para Desenvolvimento:

```kubectl create -f https://k8s.io/examples/admin/namespace-dev.json``` 

Criando o Namespace para a Produção:

```kubectl create -f https://k8s.io/examples/admin/namespace-prod.json``` 


## Trabalhando com Deployments

### Criando um Deployment

A seguir, é apresentado um exemplo de implantação. Ele cria um ReplicaSet para exibir três Pods nginx:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

Vamos aplicar esse Deployment no Namespace *Development*

```
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml --namespace=development
``` 

### Atualizando o Deployment

Neste momento que temos um aplicação rodando vamos fazer o processo de solicitar a atualização dela (através da atualização da versão de sua imagem)

```
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 --record true --namespace=development
```

Para ver o *rollout* acontecendo:

``` 
kubectl rollout status deployment.v1.apps/nginx-deployment --namespace=development
```

Para checar detales da nova versão:

```
kubectl describe deployments --namespace=development
```
### Fazendo um Rolling Back de Atualização

Atrés do *Rollout History* é possivel checar o historico de Rollouts solicitados: 

```
kubectl rollout history deployment.v1.apps/nginx-deployment --namespace=development
```

Vendo Detalhes de uma Determinada revisão:

```
kubectl rollout history deployment.v1.apps/nginx-deployment --revision=1 --namespace=development
```

Rollback para um revisão especifica com *--to-revision*

```
kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=1 --namespace=development
```

## Acessando uma Aplicação

Vamos criar um Serviço para uma aplicação executando em 5 pods

```
kubectl apply -f https://k8s.io/examples/service/load-balancer-example.yaml --namespace=production
```

Vamos ver detalhes do Deployment

```
kubectl get deployments hello-world --namespace=production
kubectl describe deployments hello-world --namespace=production
```

Vamos criar o Serviço para o Deployment

```
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service --namespace=production
```

Vamos ver detalhes do Serviço

```
kubectl get services my-service --namespace=production
```

Acompanhando a subida dos Pods

```
kubectl get pods --output=wide  --namespace=production
```

Acessando o Serviço

```
curl http://<external-ip>:<port>
```


### Efetuando o Cleanup

```
kubectl delete services my-service --namespace=production
kubectl delete deployment hello-world --namespace=production
```

