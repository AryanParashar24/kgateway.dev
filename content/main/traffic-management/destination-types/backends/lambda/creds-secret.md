---
title: Access AWS Lambda with a credentials secret
weight: 10
---

Use kgateway to route traffic requests directly to an [Amazon Web Services (AWS) Lambda](https://aws.amazon.com/lambda/resources/) function.

Note that this guide uses a Kubernetes secret that contains you AWS access key and secret key to invoke Lambda functions. To use AWS IAM roles to control access instead, see [Access AWS Lambda with a service account](/docs/traffic-management/destination-types/backends/lambda/service-accounts/) instead.

## Before you begin

{{< reuse "docs/snippets/prereq.md" >}}

## Create an AWS credentials secret

Create a Kubernetes secret that contains your AWS access key and secret key. Kgateway uses this secret to connect to AWS Lambda for authentication and function invocation.

1. Get the access key and secret key for your AWS account. Note that your [AWS credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) must have the appropriate permissions to interact with AWS Lambda.

2. Create a Kubernetes secret that contains the AWS access key and secret key.
   ```yaml
   kubectl apply -n kgateway-system -f - << EOF
   apiVersion: v1
   kind: Secret
   metadata:
     name: aws-creds
   stringData:
     accessKey: ${AWS_ACCESS_KEY_ID}
     secretKey: ${AWS_SECRET_ACCESS_KEY}
     sessionToken: ""
   type: Opaque
   EOF
   ```

## Create a Lambda function

Create an AWS Lambda function to test kgateway routing.

1. Log in to the AWS console and navigate to the Lambda page.

2. Click the **Create Function** button.

3. Name the function `echo` and click **Create function**.

4. Replace the default contents of `index.mjs` with the following Node.js function, which returns a response body that contains exactly what was sent to the function in the request body.
   
   ```js
   export const handler = async(event) => {
       const response = {
           statusCode: 200,
           body: `Response from AWS Lambda. Here's the request you just sent me: ${JSON.stringify(event)}`
       };
       return response;
   };
   ```

5. Click **Deploy**.

## Create a Backend and HTTPRoute

Create kgateway `Backend` and `HTTPRoute` resources to route requests to the Lambda function.

1. In your terminal, create a Backend resource that references the Lambda secret. Update the `region` with your AWS account region, such as `us-east-1`, and update the `accountId`.
   
   ```yaml
   kubectl apply -f - <<EOF
   apiVersion: gateway.kgateway.dev/v1alpha1
   kind: Backend
   metadata:
     name: lambda
     namespace: kgateway-system
   spec:
     type: AWS
     aws:
       region: <region>
       accountId: "<account-id>"
       auth:
         type: Secret
         secretRef:
           name: aws-creds
       lambda:
         functionName: echo
   EOF
   ```

2. Create an HTTPRoute resource that references the `lambda` Backend.
   
   ```yaml
   kubectl apply -f - <<EOF
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
     name: lambda
     namespace: kgateway-system
   spec:
     parentRefs:
       - name: http
         namespace: kgateway-system
     rules:
     - matches:
       - path:
           type: PathPrefix
           value: /echo
       backendRefs:
       - name: lambda
         namespace: kgateway-system
         group: gateway.kgateway.dev
         kind: Backend
   EOF
   ```

3. Confirm that kgateway correctly routes requests to Lambda by sending a curl request to the `echo` function.
   
   {{< tabs items="Cloud Provider LoadBalancer,Port-forward for local testing" >}}
   {{% tab %}}
   ```sh
   curl $INGRESS_GW_ADDRESS:8080/echo -d '{"key1":"value1", "key2":"value2"}' -X POST
   ```
   {{% /tab %}}
   {{% tab %}}
   ```sh
   curl localhost:8080/echo -d '{"key1":"value1", "key2":"value2"}' -X POST
   ```
   {{% /tab %}}
   {{< /tabs >}}

   Example response:
   
   ```json
   {"statusCode":200,"body":"Response from AWS Lambda. Here's the request you just sent me: {\"key1\":\"value1\",\"key2\":\"value2\"}"}% 
   ```

At this point, kgateway is routing directly to the `echo` Lambda function!

## Cleanup

{{% reuse "docs/snippets/cleanup.md" %}}

1. Delete the `lambda` HTTPRoute and `lambda` Backend.
   
   ```sh
   kubectl delete HTTPRoute lambda -n kgateway-system
   kubectl delete Backend lambda -n kgateway-system
   ```

2. Delete the `aws-creds` secret.
   
   ```sh
   kubectl delete secret aws-creds -n kgateway-system
   ```

3. Use the AWS Lambda console to delete the `echo` test function.