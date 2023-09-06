# Self-Signed Certificates in Kubernetes
 
## Passos

https://artifacthub.io/packages/helm/cert-manager/cert-manager

1. Instalação do Cert-Manager
    #https://youtu.be/IQ3G8Z1myMw?t=95 <br>
    #https://cert-manager.io/docs/ <br>
    #https://cert-manager.io/docs/installation/helm/#prerequisites <br>
    #https://youtu.be/IQ3G8Z1myMw?t=211 <br>
	<br>
 
#https://artifacthub.io/packages/helm/cert-manager/cert-manager	 <br>
 #install CRD resources <br
 $ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.4/cert-manager.crds.yaml <br> <br>
 
    $ helm repo add jetstack https://charts.jetstack.io <br>
    $ helm repo update <br>

   
    #install cert-manager <br>
    $ helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --version v1.12.4  <br>  
    


2. Gerar o certificado
    #https://youtu.be/IQ3G8Z1myMw?t=240 <br>
    #gerar certificate authority <br>
    $ openssl genrsa -out ca.key 4096 <br>

	### NOTA: O Common Name deve ser o dominio desejado( padua.iff.edu.br, cordeiro.iff.edu.br, etc) <br>
	
    #Gerar o certificado crt  <br>
    $ openssl req -new -x509 -sha256 -days 3650 -key ca.key -out ca.crt <br>
    #O certificado crt deve ser importado nas máquinas windows da rede local <br>
    #na pasta trust root certificate authority <br>


3. Cluster Issuer 
    #https://youtu.be/IQ3G8Z1myMw?t=395
    #https://cert-manager.io/docs/concepts/issuer/#namespaces <br>
    #https://youtu.be/IQ3G8Z1myMw?t=527 <br>
    #converter os certificados p/ base64 <br>
    #$ cat ca.crt | base64 -w 0 <br>
    #$ cat ca.key | base64 -w 0 <br>

    #https://shocksolution.com/2018/12/14/creating-kubernetes-secrets-using-tls-ssl-as-an-example/
    #Criar o arquivo sem erros <br>
    $ kubectl create secret tls temp-tls --key="./certs/ca.key" --cert="./certs/ca.crt" <br>
    $ kubectl get secret temp-tls -o yaml > secret.yaml <br>
    $ kubectl delete secret temp-tls <br>
    $ kubectl get clusterissuer <br>


4. Configurações finais <br>
    #https://youtu.be/IQ3G8Z1myMw?t=574
    

## Diagrama de Arquitetura

```
                Cert-Manager Objects                        Nginx1 Objects

               ┌───────────────────────┐                    ┌─────────────────────────────────┐
Created CA     │ kind: Secret          │                    │                                 │
private key ──►│ name: nginx1-ca-secret│◄─────────┐         │ kind: Ingress                   │
and cert       │ tls.key: **priv key** │          │         │ name: nginx1-ingress            │
               │ tls.crt: **cert**     │          │         │ tls:                            │
               └───────────────────────┘          │         │   - hosts:                      │
                                                  │         │     - nginx1.clcreative.home    │
               ┌──────────────────────────────┐   │    ┌────┼───secretName: nginx1-tls-secret │
               │                              │   │    │    │                                 │
               │ kind: ClusterIssuer          │   │    │    └─────────────────────────────────┘
           ┌───┤►name: nginx1-clusterissuer   │   │    │
           │   │ secretName: nginx1-ca-secret─┼───┘    │
           │   │                              │        │
           │   └──────────────────────────────┘        │
           │                                           │
           │   ┌───────────────────────────────┐       │
           │   │                               │       │
           │   │ kind: Certificate             │       │
           │   │ name: nginx1-cert             │       │
           └───┼─issuerRef:                    │       │
               │   name: nginx1-clusterissuer  │       │
               │   kind: ClusterIssuer         │       │
               │ dnsNames:                     │       │
               │   - nginx1.clcreative.home    │       │
           ┌───┼─secretName: nginx1-tls-secret │       │
           │   │                               │       │
           │   └──────────┬────────────────────┘       │
           │              │                            │
           │              │ will be created            │
           │              ▼ and managed automatically  │
           │   ┌───────────────────────────────┐       │
           │   │                               │       │
           │   │ kind: Secret                  │       │
           └───┤►name: nginx1-tls-secret◄──────┼───────┘
               │                               │
               └───────────────────────────────┘
```



### Repositórios de Referência
https://github.com/xcad2k
https://github.com/xcad2k/videos/tree/main/self-signed-certificates-in-kubernetes
