# cert-manager
Explains about how cert-manager works

- At a high level, cert-manager is a Kubernetes controller that watches for Certificate resource objects in the Kubernetes API server. When a new Certificate object is created or updated, cert-manager will use the issuer specified in the Certificate object to issue a new certificate.

- Here are the steps involved in how cert-manager works behind the scenes:

  - Watch for Certificate resources: Cert-manager listens for changes to Certificate resources in the Kubernetes API server. When a new Certificate object is created, updated or deleted, cert-manager will detect the change and process it.

  - Verify the Issuer or ClusterIssuer: When a new Certificate object is created, cert-manager first verifies that the Issuer or ClusterIssuer specified in the Certificate object is valid and authorized to issue certificates.

  - Request a certificate: Once cert-manager has verified the Issuer or ClusterIssuer, it will create a Certificate Signing Request (CSR) for the certificate. The CSR contains information about the certificate, such as the Common Name and the Subject Alternative Names.

  - Validate ownership of the domain: Before issuing the certificate, cert-manager must verify that the entity requesting the certificate has ownership of the domain. To do this, cert-manager will use one of the supported ACME challenges (HTTP-01, DNS-01 or TLS-ALPN-01) to validate ownership of the domain.

  - Issue the certificate: Once cert-manager has validated ownership of the domain, it will issue the certificate using the Issuer or ClusterIssuer specified in the Certificate object. The certificate is then stored in a Kubernetes secret.

  - Renew the certificate: Certificates have a limited lifespan and must be renewed before they expire. Cert-manager monitors the expiration date of the certificate and automatically initiates the renewal process when necessary.

  - Overall, cert-manager simplifies the process of managing and issuing TLS certificates in Kubernetes. By automating the process of requesting, validating, and renewing certificates, cert-manager makes it easy to secure your Kubernetes applications with TLS encryption.

- Cert-manager is a Kubernetes add-on that automates the management and issuance of TLS certificates. It uses the Kubernetes API to issue, renew, and revoke certificates. Here are the steps involved in using cert-manager:

  - Install cert-manager: You can install cert-manager using a Kubernetes manifest or by using a package manager like Helm.

  - Create a ClusterIssuer or an Issuer: A ClusterIssuer or an Issuer is a Kubernetes resource that specifies how certificates should be issued. An Issuer is scoped to a single namespace, while a ClusterIssuer is scoped to the entire cluster. A ClusterIssuer can be used by multiple namespaces.

  - Create a Certificate resource: A Certificate resource is a Kubernetes resource that defines the properties of a certificate, such as the Common Name (CN), DNS names, IP addresses, and the Issuer or ClusterIssuer to use. You can create a Certificate resource using a YAML file or using kubectl.

  - Verify the Certificate resource: After creating the Certificate resource, cert-manager will automatically attempt to issue a certificate. You can verify the status of the Certificate resource using kubectl.

  - Use the certificate: Once the certificate has been issued, you can use it in your Kubernetes resources, such as Ingress, by referencing it using its name and namespace.

# Here is an example YAML file for creating a ClusterIssuer:


```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - http01:
        ingress:
          class: nginx
 ```
          
In this example, the ClusterIssuer specifies that Let's Encrypt's staging environment should be used to issue certificates. The email field specifies the email address to use for registration and renewal notifications. The privateKeySecretRef field specifies the name of a Kubernetes secret that will store the private key used for the certificate. Finally, the solvers field specifies that HTTP01 challenge should be used with the Nginx ingress class to validate ownership of the domain.

# Here is an example YAML file for creating a Certificate:

```yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: example-com
spec:
  secretName: example-com-tls
  issuerRef:
    name: letsencrypt-staging
  commonName: example.com
  dnsNames:
  - example.com
  - www.example.com
  
 ```
  
- In this example, the Certificate specifies the name of a Kubernetes secret that will store the certificate and private key, the Issuer to use, the Common Name and DNS names for the certificate. The Issuer reference specifies the name of the ClusterIssuer created earlier.

  - When using the DNS-01 challenge with cert-manager, you can use various DNS providers to manage your DNS records, including AWS Route53.

  - Here are the steps involved in configuring cert-manager with AWS Route53 to use the DNS-01 challenge:

  - Create an IAM user: First, you need to create an IAM user in your AWS account with the appropriate permissions to manage Route53 DNS records. The required permissions include the following:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets",
                "route53:GetChange"
            ],
            "Resource": [
                "arn:aws:route53:::hostedzone/*"
            ]
        }
    ]
}

```
 - Create an IAM Access Key and Secret Access Key: Next, you need to create an Access Key and Secret Access Key for the IAM user. You will use these keys in your cert-manager configuration.

 - Configure a Kubernetes Secret: You need to create a Kubernetes Secret to store the AWS Access Key and Secret Access Key. The Secret should have the following format:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: route53-secret
  namespace: cert-manager
type: Opaque
data:
  access-key-id: <base64-encoded-access-key-id>
  secret-access-key: <base64-encoded-secret-access-key>
```

# You can use the base64 command to encode the Access Key and Secret Access Key.

- Configure a ClusterIssuer: You can create a ClusterIssuer to use the AWS Route53 DNS provider with the DNS-01 challenge. Here is an example ClusterIssuer configuration:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-staging
    solvers:
    - dns01:
        route53:
          accessKeyID: access-key-id
          secretAccessKeySecretRef:
            name: route53-secret
            key: secret-access-key
 ```
 
- In this example, the ClusterIssuer specifies Let's Encrypt's staging environment and uses the DNS-01 challenge with the Route53 DNS provider. The accessKeyID field specifies the AWS Access Key ID, and the secretAccessKeySecretRef field references the Kubernetes Secret created earlier.

- Create a Certificate: Finally, you can create a Certificate resource that references the ClusterIssuer created earlier. Cert-manager will use the ClusterIssuer to issue the certificate using the DNS-01 challenge with the Route53 DNS provider.

