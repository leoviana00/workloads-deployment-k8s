## Instale o Jenkins com o Helm v3

- Jenkins no Kubernetes

## Instalar Helm

## Configurar Helm

```bash
helm repo add jenkinsci https://charts.jenkins.io
helm repo update
kubectl create ns jenkins 
```

- Listar o Helm
```bash
helm search repo jenkinsci
```

## CCriar um volume persistent
- Criar um volume chamado jenkins-pv

```bash
kubectl apply -f jenkins-volume.yaml
```

- OBS: o hostPath usa o /data/jenkins-volume/ do seu nó para emular o armazenamento conectado à rede. Essa abordagem é adequada apenas para fins de desenvolvimento e teste. Para produção, você deve fornecer um recurso de rede, como um disco permanente do Google Compute Engine ou um volume do Amazon Elastic Block Store.

- Alterar manualmente as permissões para permitir que a conta jenkins grave seus dados.

```bash
sudo chown -R 1000:1000 /data/jenkins-volume
```

## Criar uma conta de serviço
- Criar uma conta de serviço chamada jenkins:

- Um ClusterRole é um conjunto de permissões que podem ser atribuídas a recursos em um determinado cluster. As APIs do Kubernetes são categorizadas em grupos de APIs, com base nos objetos de API aos quais elas se relacionam. Ao criar um ClusterRole, você pode especificar as operações que podem ser executadas pelo ClusterRole em um ou mais objetos de API em um ou mais grupos de API, assim como fizemos acima. ClusterRoles tem vários usos. Você pode usar um ClusterRole para:

```console
definir permissões em recursos com namespace e ser concedidas em namespaces individuais
definir permissões em recursos com namespace e ser concedida em todos os namespaces
definir permissões em recursos com escopo de cluster
```
- Se você deseja definir uma função em todo o cluster, use um ClusterRole; se você quiser definir uma função em um namespace, use uma função.

- Uma associação de função concede as permissões definidas em uma função a um usuário ou conjunto de usuários. Ele contém uma lista de assuntos (usuários, grupos ou contas de serviço) e uma referência à função que está sendo concedida.

- Um RoleBinding pode fazer referência a qualquer Role no mesmo namespace. Como alternativa, um RoleBinding pode fazer referência a um ClusterRole e associar esse ClusterRole ao namespace do RoleBinding. Para vincular um ClusterRole a todos os namespaces em nosso cluster, usamos um ClusterRoleBinding.

```bash
kubectl apply -f jenkins-sa.yaml
```

## Instalar Jenkins

- Para habilitar a persistência, criaremos um arquivo de substituição e o passaremos como um argumento para a CLI do Helm utilizando o jenkins-values.yaml. O jenkins-values.yamlé usado como um modelo para fornecer os valores necessários para a configuração.

```bash
helm install jenkins -n jenkins -f jenkins-values.yaml jenkinsci/jenkins
```
## Obtenha sua senha de usuário 'admin' executando:
```bash
jsonpath="{.data.jenkins-admin-password}"
secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
echo $(echo $secret | base64 --decode)
```

- user: admin
- senha: bFrtGOrcpju2CD9Qa78265

## Referência
- https://www.jenkins.io/doc/book/installing/kubernetes/