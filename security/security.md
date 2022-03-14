Pre-Requisite:

- Create a Linux container / Linux server were we can run openssl commands.

yum -y install openssl nano

- Connect the linux box with existing K8s setup.

Kubectl Installation:

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl


Step 1: Create a new private key  and CSR

openssl genrsa -out zeal.key 2048
openssl req -new -key zeal.key -out zeal.csr -subj "/CN=zeal/O=kplabs"

Step 2: Encode the csr

cat zeal.csr | base64 | tr -d '\n'

Step 3: Generate the Kubernetes Signing Request

apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: zeal-csr
spec:
  groups:
  - system:authenticated
  request: 
  usages:
  - digital signature
  - key encipherment
  - client auth


Step 4: Apply the Signing Requests:



kubectl apply -f signingrequest.yaml



Step 5: Approve the csr



kubectl certificate approve zeal-csr



Step 6: Download the Certificate from csr



kubectl get csr zeal-csr -o jsonpath='{.status.certificate}' | base64 -d > zeal.crt



Step 7: Create a new context



kubectl config set-credentials zeal --client-certificate=zeal.crt --client-key=zeal.key



Step 8: Set new Context



kubectl config set-context zeal-context --cluster do-blr1-kplabs-k8s --user=zeal



Step 9: Use Context to Verify



kubectl --context=zeal-context get pods