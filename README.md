# Crossplane

> https://github.com/yesteph/crossplane-demo

## Install

> [crossplane setup](https://crossplane.io/docs/v1.2/getting-started/install-configure.html)
```
curl -sL https://raw.githubusercontent.com/crossplane/crossplane/master/install.sh | sh
```

> [crossplane provider setup](https://crossplane.io/docs/v1.2/cloud-providers/aws/aws-provider.html)
> [a gist to compare crossplane %% install](https://gist.github.com/vfarcic/b992aeb8bbee2df1823c45475ceb4373)
```
helm repo add crossplane-stable \
    https://charts.crossplane.io/stable

helm repo update

helm upgrade --install \
    crossplane crossplane-stable/crossplane \
    --namespace crossplane-system \
    --create-namespace \
    --wait
```
### Setup AWS Provider Manually # This wont work with vault create a file manually

[Installing Providers]([https://](https://crossplane.io/docs/v1.2/concepts/providers.html))

```
cat > provider.yaml << EOF
---
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-aws
spec:
  package: "crossplane/provider-aws:v0.19.0"
EOF
  
```

> Be sure you have such a file to create secret with aws-vault you need to do it manually
```
cat > creds.conf <<EOF
---
[demo-cr]
aws_access_key_id = XXXXXXXXXXXXXXXXXXXXXXXXXX
aws_secret_access_key = XXXXXXXXXXXXXXXXXXXXXXXXXX
EOF

export BASE64ENCODED_AWS_ACCOUNT_CREDS=$(echo $(cat creds.conf) | base64 | tr -d "\n")
```
```
cat > providerconfig.yaml <<EOF
---
apiVersion: v1
kind: Secret
metadata:
  name: aws-account-creds
  namespace: crossplane-system
type: Opaque
data:
  credentials: ${BASE64ENCODED_AWS_ACCOUNT_CREDS}
---
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-account-creds
      key: credentials
EOF
```

### apply it to the cluster:
```
kubectl apply -f provider.yaml
kubectl apply -f providerconfig.yaml
```

#### delete the credentials variable
```
unset BASE64ENCODED_AWS_ACCOUNT_CREDS
```


## DIFFERENT SUBJECT - UTIL FOR DOCKER IMAGE
### A tool for exploring a docker image, layer contents, and discovering ways to shrink the size of your Docker/OCI image.
https://github.com/wagoodman/dive
