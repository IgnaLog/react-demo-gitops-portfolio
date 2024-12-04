argo: installargo loginargo addcluster createns project installapp
argo-iu: installargo loginargo addcluster createns secret project installimageupdater installapp

addcluster:
	@echo "| > AÑADIMOS EL CLUSTER AKS A ARGOCD"
	yes | argocd cluster add aks-demo --name aks-demo-cluster

createns:
	@echo "| > CREACION EL NAMESPACE argocd-demo"
	kubectl create ns argocd-demo

project:
	@echo "| > CREACION DE PROYECTO DEMO"
	kubectl apply -f ../manifiests/argocd/project-argocd-demo.yaml


#---------------------------------------------------------------------------- argocd-iu
