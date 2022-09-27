# knative-minikube
Knative on macOS Apple M1 without Docker for Mac / Windows but with qemu as a Docker replacement + ZSH

## zsh-completions

```bash
brew reinstall zsh-completions

echo '
if type brew &>/dev/null; then
  FPATH=$(brew --prefix)/share/zsh-completions:$FPATH
  autoload -Uz compinit
  compinit
fi
' >> ~/.zshrc

source ~/.zshrc
```

## qemu

```bash
brew reinstall qemu

echo '
if type brew &>/dev/null; then
  FPATH=$(brew --prefix)/Cellar/qemu/*/bin:$FPATH
  PATH=$(brew --prefix)/Cellar/qemu/*/bin:$PATH
  autoload -Uz compinit
  compinit
fi
' >> ~/.zshrc

sudo mkdir -p /opt/homebrew/Cellar
sudo chown -Rfv $(whoami) /opt/homebrew
ln -s $(brew --prefix)/Cellar/qemu /opt/homebrew/Cellar/qemu 

source ~/.zshrc
```

## minikube

```bash
brew reinstall minikube

minikube completion zsh > ~/.minikube-completion
echo '[ -s "$HOME/.minikube-completion" ]] && source $HOME/.minikube-completion' >> ~/.zshrc

source ~/.zshrc
```

## start

```bash
minikube start --driver=qemu2
```

## kubectl

```bash
minikube kubectl
kubectl get --all-namespaces pods      -w
```

## knative

### install

```bash
brew reinstall kn
kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.7.1/operator.yaml
```

### verify

```bash
kubectl get deployment knative-operator
```

### logs

```bash
kubectl logs -f deploy/knative-operator
```

### networking

```bash
kubectl create namespace knative-serving
kubectl get ns

kubectl config set-context --current --namespace=knative-serving

echo '
apiVersion: operator.knative.dev/v1beta1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  # ...
  ingress:
    kourier:
      enabled: true
  config:
    network:
      ingress-class: "kourier.ingress.networking.knative.dev"
' > /tmp/kn.networking.txt && kubectl apply -f /tmp/kn.networking.txt

kubectl get -n knative-serving service kourier
kubectl get -n knative-serving deployment

kubectl get -n knative-serving KnativeServing knative-serving      -w
#NAME              VERSION   READY   REASON
#knative-serving   1.7.1     False
#knative-serving   1.7.1     True

kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.7.2/serving-default-domain.yaml 
```

### eventing

```bash
echo '
apiVersion: v1
kind: Namespace
metadata:
  name: knative-eventing
---
apiVersion: operator.knative.dev/v1beta1
kind: KnativeEventing
metadata:
  name: knative-eventing
  namespace: knative-eventing
' > /tmp/kn.eventing.txt && kubectl apply -f /tmp/kn.eventing.txt

kubectl get deployment -n knative-eventing

kubectl get KnativeEventing knative-eventing -n knative-eventing      -w
#NAME               VERSION   READY   REASON
#knative-eventing   1.7.2     False   NotReady # <-- TODO
```

## cleanup

```bash
kubectl delete KnativeEventing knative-eventing -n knative-eventing
kubectl delete KnativeServing knative-serving -n knative-serving
kubectl delete -f https://github.com/knative/operator/releases/download/knative-v1.7.1/operator.yaml

minikube stop --all=true
minikube delete --all=true
```

## RTFM

* https://www.youtube.com/watch?v=LGNEG-t96eE&ab_channel=DevOpsToolkit
* https://redhat-developer-demos.github.io/knative-tutorial/knative-tutorial/setup/minikube.html
* https://minikube.sigs.k8s.io/docs/commands/completion/
* https://knative.dev/docs/install/operator/knative-with-operators/
* https://flaviocopes.com/linux-command-xargs/#:~:text=The%20xargs%20command%20is%20used,the%20input%20of%20another%20command.
