# gloo-lab
Testing gloo scenarios

# Steps for setup

* If you don't already have the gloo image on machine install it:

```shell
helm repo add gloo https://storage.googleapis.com/solo-public-helm
helm repo update
```

* Configure app in Auth0, gather client id and secret
* Create secret for client_secret

```bash
echo -n "your-auth0-client-secret" | base64
```

`gloo-config/auth0-secret.yaml`
```yaml 
apiVersion: v1
kind: Secret
metadata:
  name: auth0-secret
  namespace: gloo-system
type: Opaque
data:
  client-secret: <base64-encoded-client-secret>
```

* K8s running in Rancher desktop or whatever as long as helm and kubectl works.

```shell
helm install gloo gloo/gloo --namespace gloo-system --create-namespace
kubectl apply -f gloo-config/auth0-secret.yaml
kubectl apply -f gloo-config/authconfig.yaml
kubectl apply -f gloo-config/upstream.yaml
kubectl apply -f gloo-config/virtualservice.yaml
kubectl apply -f k8s/frontend.yaml
kubectl apply -f k8s/backend.yaml
```

## Note on Auth

You'll notice that even with the oauth configured, there is no prompting for credentials in development, this is due to the
External Auth API being an enterprise-only licensed feature for Gloo.  You can see it when describing the authconfig:

```shell
gloo-lab git:(main) ✗ k describe authconfig auth0-oauth --namespace gloo-system
Name:         auth0-oauth
Namespace:    gloo-system
Labels:       <none>
Annotations:  <none>
API Version:  enterprise.gloo.solo.io/v1
Kind:         AuthConfig
Metadata:
  Creation Timestamp:  2025-04-14T18:47:29Z
  Generation:          2
  Resource Version:    1375
  UID:                 e96823ca-3eef-448a-b670-783ab325c888
Spec:
  Configs:
    oauth2:
      Oidc Authorization Code:
        App URL:        http://localhost
        Callback Path:  /callback
        Client Id:      super-secret
        Client Secret Ref:
          Name:       auth0-secret
          Namespace:  gloo-system
        Issuer URL:   https://auth0url.com/
        Scopes:
          openid
          profile
          email
Status:
  Statuses:
    Gloo - System:
      Reason:  1 error occurred:
               * The Gloo Advanced Extauth API is an enterprise-only feature, please upgrade or use the Envoy Extauth API instead


      Reported By:  gloo
      State:        Rejected
Events:             <none>
```