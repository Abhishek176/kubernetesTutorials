Pre-Requisite:
- Create a Linux container / Linux server were we can run openssl commands.
yum -y install openssl vi

- Connect the linux box with existing K8s setup.

Kubectl Installation:

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

Step 1: Create a new private key  and CSR
openssl genrsa -out abhishek.key 2048
openssl req -new -key abhishek.key -out abhishek.csr -subj "/CN=abhishek"

Step 2: Encode the csr
cat abhishek.csr | base64 | tr -d '\n'

Step 3: Generate the Kubernetes Signing Request
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: abhishek-csr
spec:
  groups:
  - system:authenticated
  request: $(cat abhishek.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - digital signature
  - key encipherment
  - client auth

Step 4: Apply the Signing Requests:
kubectl apply -f signingrequest.yaml

Step 5: Approve the csr
kubectl certificate approve abhishek-csr

Step 6: Download the Certificate from csr
kubectl get csr abhishek-csr -o jsonpath='{.status.certificate}' | base64 -d > abhishek.crt

Step 7: Create a new context
kubectl config set-credentials abhishek --client-certificate=abhishek.crt --client-key=abhishek.key

Step 8: Set new Context
kubectl config set-context abhishek-context --cluster cluster_name --user=abhishek

Step 9: Use Context to Verify
kubectl --context=abhishek-context get pods