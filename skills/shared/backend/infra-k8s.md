# Infrastructure - Kubernetes

[[_TOC_]]

## Уровни навыка
### Уровень 1

Обязательно:

* Знает концепции, основных абстракций k8s: namespace, deployment, replica set, pod, service, ingress, configmap
* Знает что такое пробы, какие бывают виды, от чего зависит статус
* Знает про управление ресурсами (request, limit)

Желательно:

* Умеет общаться с кубом при помощи kubectl
* Знает жизненный цикл pod'ов
* Знает про сущность cron jobs
* Знает про StatefulSet
* Умеет локально развернуть ресурсы (minikube)

### Уровень 2

Обязательно:

* Умеет пользоваться какими-нибудь инструментами для работы с ресурсами k8s as a code
  * helm
  * helmfile
  * kustomize
  * kapitan
* Умеет реализовывать разные стратегии деплоймента (rolling update, canary)

Желательно:

* Знает как устроена сеть в кубе
* Знает архитектуру куба
* Знает как профилировать и снимать дампы в кубе

## Метод оценки

TODO

## Как прокачать
* The Kubernetes Book by Nigel Poulton
* Kubernetes in Action by Marko Luksa
* Kubernetes Patterns by Bilgin Ibryam, Roland Huß