Create Namespace
kubectl create namespace argocd

Install Argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.11.8/manifests/install.yaml

Install jq if not already
sudo apt  install jq

Get the admin password for ArgoCD
kubectl get secret argocd-initial-admin-secret -n argocd -o json | jq .data.password -r | base64 -d

Edit the service, change the type to NodePort and add desired nodePort number under -name: https
kubectl edit svc argocd-server -n argocd

Installing ArgoCD cli
Download the binary wherever you have write permissions
curl -sSL -o ~/argocd https://github.com/argoproj/argo-cd/releases/download/v2.4.11/argocd-linux-amd64
Give Execute permission
chmod +x ~/argocd
Move it to bin directory so that it gets added to $PATH variable of your system
sudo mv ~/argocd /usr/local/bin/
You can check the version
argocd version

You need to login into your ArgoCd server via CLI to perform further commands
argocd login <node-name:port>
username
password

Create your first app via CLI, which will be added in default Project with no restrictions, Use the following command and replace your git repository
You need to create namespace 'solar-systrem' for the app as well, or use an existing namespace
argocd app create solar-system-app-2 \
--repo https://github.com/sidd-harth/gitops-argocd \
--path ./solar-system \
--dest-namespace solar-system \
--dest-server https://kubernetes.default.svc

You have to sync the app manually, since we have chosen it to be manual for now
Synchronize the solar-system-app-2 application using Argocd CLI.
argocd app sync solar-system-app-2

You can check the app is running or not in your browser http://<node-name>:port

To change the reconciliation time to 1 minute for ArgoCD which is curently 3 minutes by default
kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"timeout.reconciliation":"60s"}}'
or change in the conif map as below
data:
  timeout.reconciliation: 60s

Otherwise if you want to avoid delay, you can configure a webook from your git repository, but make sure you need to have a secure certificate for ArgoCd if you want to use webhook else you can set the insecure flag in the argo cd deployment as shown below:
- command:
  - argocd-server
  - --insecure

Webhook can be added by going int settings of git platform and adding the url of argocd into TargetUrl/Payload field for sending POST requests of push events:
<argo-cd-url>/api/webhook