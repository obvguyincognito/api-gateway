# Expose and secure a workload with JWT

This tutorial shows how to expose and secure Services using APIGateway Controller. The Controller reacts to an instance of the APIRule custom resource (CR) and creates an Istio VirtualService and [Oathkeeper Access Rules](https://www.ory.sh/docs/oathkeeper/api-access-rules) according to the details specified in the CR. To interact with the secured workloads, the tutorial uses a JWT token.

## Prerequisites

* Deploy a [sample HttpBin Service](../01-00-create-workload.md).
* [JSON Web Token (JWT)](./01-51-get-jwt.md)
* Set up [your custom domain](../01-10-setup-custom-domain-for-workload.md) or use a Kyma domain instead. 
* Depending on whether you use your custom domain or a Kyma domain, export the necessary values as environment variables:
  
  <div tabs name="export-values">

    <details>
    <summary>
    Custom domain
    </summary>
    
    ```bash
    export DOMAIN_TO_EXPOSE_WORKLOADS={DOMAIN_NAME}
    export GATEWAY=$NAMESPACE/httpbin-gateway
    ```
    </details>

    <details>
    <summary>
    Kyma domain
    </summary>

    ```bash
    export DOMAIN_TO_EXPOSE_WORKLOADS={KYMA_DOMAIN_NAME}
    export GATEWAY=kyma-system/kyma-gateway
    ```
    </details>
  </div>  

## Expose, secure, and access your workload

1. Expose the Service and secure it by creating an APIRule CR in your Namespace. Run:

   ```bash
   cat <<EOF | kubectl apply -f -
   apiVersion: gateway.kyma-project.io/v1beta1
   kind: APIRule
   metadata:
     name: httpbin
     namespace: $NAMESPACE
   spec:
     host: httpbin.$DOMAIN_TO_EXPOSE_WORKLOADS   
     service:
       name: $SERVICE_NAME
       port: 8000
     gateway: $GATEWAY
     rules:
       - accessStrategies:
         - handler: jwt
           config:
             jwks_urls:
             - $JWKS_URI
         methods:
           - GET
         path: /.*
   EOF
   ```

   >**NOTE:** If you are using k3d, add `httpbin.kyma.local` to the entry with k3d IP in your system's `/etc/hosts` file.

2. To access the secured Service, call it using the JWT access token:

   ```bash
   curl -ik https://httpbin.$DOMAIN_TO_EXPOSE_WORKLOADS/headers -H "Authorization: Bearer $ACCESS_TOKEN"
   ```

  If successful, the call returns the code `200 OK` response.
