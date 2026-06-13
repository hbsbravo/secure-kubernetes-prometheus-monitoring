\# Secret Management with Kubernetes Sealed Secrets



\## Overview



This project removes plaintext secrets from the Kubernetes deployment configuration and replaces them with a sealed secret workflow. The goal is to prevent sensitive values from being exposed in source control while still allowing the application and database containers to receive required secrets at runtime.



\## Sensitive Values Protected



The deployment originally required sensitive values such as:



\* Django `SECRET\_KEY`

\* MySQL root password



These values should not be hardcoded in Kubernetes deployment files or committed directly to GitHub.



\## Application Configuration Change



The Django application was updated so the secret key is loaded from an environment variable:



```python

SECRET\_KEY = os.environ\["SECRET\_KEY"]

```



This allows Kubernetes to inject the secret value into the running container instead of storing it directly in the application source code.



\## Kubernetes Secret Handling



A temporary raw Kubernetes secret file was created locally and then encrypted into a SealedSecret using `kubeseal`.



The raw secret file was intentionally excluded from source control:



```text

django-secret-raw.yaml

```



The sealed version is safe to keep in the repository:



```text

module2.yaml

```



\## Deployment Configuration Changes



The Django deployment was updated so sensitive environment variables are pulled from a Kubernetes secret using `secretKeyRef`.



The MySQL deployment was also updated so the root password is provided by the Kubernetes secret instead of being stored directly in the deployment YAML.



Non-sensitive configuration values, such as database hostnames and allowed hosts, remain as normal environment variables.



\## Validation Steps



The Kubernetes resources were applied locally using `kubectl`, and the Django and MySQL pods were restarted so they could consume the updated secret-based configuration.



Validation included checking:



```bash

kubectl get pods -A

kubectl get sealedsecrets -A

kubectl get secrets -A

```



The expected result is that:



\* The Django pod is running.

\* The MySQL pod is running.

\* The Sealed Secrets controller is running.

\* The sealed secret exists.

\* The unsealed Kubernetes secret exists in the cluster.



\## Security Value



This change improves the deployment by:



\* Removing plaintext secrets from Kubernetes YAML files.

\* Preventing accidental secret exposure in GitHub.

\* Supporting secure runtime configuration through Kubernetes secrets.

\* Preserving a reproducible deployment process using Sealed Secrets.



