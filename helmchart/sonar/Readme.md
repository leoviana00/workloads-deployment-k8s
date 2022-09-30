<h1 align="center">Sonarqube Helm Chart</h1>

<p align="center">
  <img alt="sonar-helm" src="../../images/sonar-helm.png">
</p>

## Repository

```bash
helm repo add sonarqube https://SonarSource.github.io/helm-chart-sonarqube
helm repo update
kubectl create namespace sonarqube
helm upgrade --install -values.yaml -n sonarqube sonarqube sonarqube/sonarqube
```
- Personalizar o `values`

