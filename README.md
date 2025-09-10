# Volumes in Kubernetes

https://bluexp.netapp.com/blog/cvo-blg-5-types-of-kubernetes-volumes-and-how-to-work-with-them

## Table of Contents
- [Azure Files for AKS Storage](#azure-files-for-aks-storage)
  - [Introduction](#introduction)
  - [kube-manifests-v1: Custom Storage Class](#kube-manifests-v1-custom-storage-class)
  - [kube-manifests-v2: AKS defined default storage class](#kube-manifests-v2-aks-defined-default-storage-class)
  - [Create or Review kube-manifests-v1 and Nginx Files](#create-or-review-kube-manifests-v1-and-nginx-files)
  - [Deploy Kube Manifests V1](#deploy-kube-manifests-v1)
  - [Upload Nginx Files to Azure File Share](#upload-nginx-files-to-azure-file-share)
  - [Access Application & Test](#access-application--test)
  - [Clean-Up](#clean-up)
  - [Create or Review kube-manifests-v2 and Nginx Files](#create-or-review-kube-manifests-v2-and-nginx-files)
  - [Deploy Kube Manifests V2](#deploy-kube-manifests-v2)
  - [Upload Nginx Files to Azure File Share (V2)](#upload-nginx-files-to-azure-file-share-v2)
  - [Access Application & Test (V2)](#access-application--test-v2)
  - [Clean-Up (V2)](#clean-up-v2)
- [References](#references)
- [To Create AWS EBS Volume to Attach with Kubernetes](#to-create-aws-ebs-volume-to-attach-with-kubernetes)
  - [Step 1: Check OIDC Provider Exist or Not](#step-1-check-oidc-provider-exist-or-not)
  - [Step 2: Copy the URL of OIDC](#step-2-copy-the-url-of-oidc)
  - [Step 3: Create OIDC Identity Provider](#step-3-create-oidc-identity-provider)
  - [Step 4: Create IAM Policy for EBS CSI Driver](#step-4-create-iam-policy-for-ebs-csi-driver)
  - [Step 5: Create IAM Role for EBS CSI Driver](#step-5-create-iam-role-for-ebs-csi-driver)
  - [Step 6: Associate the IAM Role with the EBS CSI Driver Service Account](#step-6-associate-the-iam-role-with-the-ebs-csi-driver-service-account)
- [To Create S3 as a Volume to Attach with Kubernetes](#to-create-s3-as-a-volume-to-attach-with-kubernetes)
  - [Step 1: Create S3 Bucket & Create the IAM Policy for S3](#step-1-create-s3-bucket--create-the-iam-policy-for-s3)
  - [Step 2: Create the IAM Role](#step-2-create-the-iam-role)
  - [Step 3: Associate the Role with a Kubernetes Service Account](#step-3-associate-the-role-with-a-kubernetes-service-account)
  - [Step 4: Install Mountpoint for Amazon S3 CSI Driver](#step-4-install-mountpoint-for-amazon-s3-csi-driver)
  - [Step 5: Check Add-ons Created or Not](#step-5-check-add-ons-created-or-not)

# Azure Files for AKS Storage

## Introduction
- Understand Azure Files
- We are going to write a Deployment Manifest for NGINX Application which will have its static content served from **Azure File Shares** in **app1** folder
- We are going to mount the file share to a specific path `mountPath: "/usr/share/nginx/html/app1"` in the Nginx container

### kube-manifests-v1: Custom Storage Class
- We will define our own custom storage class with desired permissions
  - Standard_LRS - standard locally redundant storage (LRS)
  - Standard_GRS - standard geo-redundant storage (GRS)
  - Standard_ZRS - standard zone redundant storage (ZRS)
  - Standard_RAGRS - standard read-access geo-redundant storage (RA-GRS)
  - Premium_LRS - premium locally redundant storage (LRS)

### kube-manifests-v2: AKS Defined Default Storage Class
- With default AKS created storage classes only below two options are available for us.
  - Standard_LRS - standard locally redundant storage (LRS)
  - Premium_LRS - premium locally redundant storage (LRS)

- **Important Note:** Azure Files support premium storage in AKS clusters that run Kubernetes 1.13 or higher, minimum premium file share is 100GB


## Create or Review kube-manifests-v1 and Nginx Files
- Kube Manifests
  - 01-Storage-Class.yml
  - 02-Persistent-Volume-Claim.yml
  - 03-Nginx-Deployment.yml
  - 04-Nginx-Service.yml
- nginx-files
  - file1.html
  - file2.html

- k8s Deployment manifest - core item for review
```yml
          volumeMounts:
            - name: my-azurefile-volume
              mountPath: "/usr/share/nginx/html/app1"
      volumes:
        - name: my-azurefile-volume
          persistentVolumeClaim:
            claimName: my-azurefile-pvc
```

## Deploy Kube Manifests V1
```bash
# Deploy
kubectl apply -f kube-manifests-v1/

# Verify SC, PVC, PV
kubectl get sc, pvc, pv

# Verify Pod
kubectl get pods
kubectl describe pod <pod-name>

# Get Load Balancer Public IP
kubectl get svc

# Access Application
http://<External-IP-from-get-service-output>
http://<External-IP-from-get-service-output>/app1/index.html
```

## Upload Nginx Files to Azure File Share
- Go to Storage Accounts
- Select and Open storage account under resource group **mc_aks-rg1_aksdemo1_eastus**
- In **Overview**, go to **File Shares**
- Open File share with name which starts as **kubernetes-dynamic-pv-xxxxxx**
- Click on **Upload** and upload
  - file1.html
  - file2.html

## Access Application & Test
```bash
# URLs
http://<External-IP-from-get-service-output>/app1/file1.html
http://<External-IP-from-get-service-output>/app1/file2.html
```

## Clean-Up
```bash
# Delete
kubectl delete -f kube-manifests-v1/
```

## Create or Review kube-manifests-v2 and Nginx Files
- Kube Manifests
  - 01-Persistent-Volume-Claim.yml
  - 02-Nginx-Deployment.yml
  - 03-Nginx-Service.yml
- nginx-files
  - file1.html
  - file2.html

## Deploy Kube Manifests V2
```bash
# Deploy
kubectl apply -f kube-manifests-v2/

# Verify SC, PVC, PV
kubectl get sc, pvc, pv

# Verify Pod
kubectl get pods
kubectl describe pod <pod-name>

# Get Load Balancer Public IP
kubectl get svc

# Access Application
http://<External-IP-from-get-service-output>
```

## Upload Nginx Files to Azure File Share (V2)
- Go to Storage Accounts
- Select and Open storage account under resource group **mc_aks-rg1_aksdemo1_eastus**
- In **Overview**, go to **File Shares**
- Open File share with name which starts as **kubernetes-dynamic-pv-xxxxxx**
- Click on **Upload** and upload
  - file1.html
  - file2.html

## Access Application & Test (V2)
```bash
# URLs
http://<External-IP-from-get-service-output>/app1/file1.html
http://<External-IP-from-get-service-output>/app1/file2.html
```

## Clean-Up (V2)
```bash
# Delete
kubectl delete -f kube-manifests-v2/
```

## References
- https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv

## To Create AWS EBS Volume to Attach with Kubernetes

Open the link and follow the instructions
- https://g.co/gemini/share/29240b42665c

OR follow these steps:

## Step 1: Check OIDC Provider Exist or Not
Run this command to check the OIDC provider
```bash
aws eks describe-cluster --name <your-cluster-name> --region <your-region> --query "cluster.identity.oidc.issuer" --output text
```

## Step 2: Copy the URL of OIDC
Copy the URL of OIDC, which looks like this: https://oidc.eks.us-east-1.amazonaws.com/id/6BB4C067DC5EBE722ED268A6DD3D8632

## Step 3: Create OIDC Identity Provider

- Go to IAM in AWS console and click on Identity Providers and then click on Add Provider
- Select Provider Type as OIDC
- Paste the URL which you copied in Step 2
- For Audience, enter sts.amazonaws.com
- Click on Add Provider

## Step 4: Create IAM Policy for EBS CSI Driver

- Go to IAM policy section and click on create policy
- Click on JSON tab and paste the below code
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:AttachVolume",
                "ec2:CreateSnapshot",
                "ec2:CreateTags",
                "ec2:CreateVolume",
                "ec2:DeleteSnapshot",
                "ec2:DeleteTags",
                "ec2:DeleteVolume",
                "ec2:DescribeAvailabilityZones",
                "ec2:DescribeInstances",
                "ec2:DescribeSnapshots",
                "ec2:DescribeTags",
                "ec2:DescribeVolumes",
                "ec2:DescribeVolumesModifications"
            ],
            "Resource": "*"
        }
    ]
}
```
- Click on Review policy
- Enter the name as MyEBSCSIPolicy
- Click on Create policy

## Step 5: Create IAM Role for EBS CSI Driver

- Go to IAM role section and click on Create role
- Select Web identity
- Select the OIDC provider which you created in Step 3
- For Audience, enter sts.amazonaws.com
- Click on Next: Permissions
- Search for the policy which you created in Step 4 (MyEBSCSIPolicy)
- Select the policy and click on Next: Tags
- Click on Next: Review
- Enter the Role name as MyEBSCSIRole
- Click on Create role

## Step 6: Associate the IAM Role with the EBS CSI Driver Service Account

The final step is to tell Kubernetes to use this IAM role with the EBS CSI driver's service account.

First, you need to find the name of the service account used by the EBS CSI driver. It's typically named ebs-csi-controller-sa and is in the kube-system namespace. You can verify this with the command:
```bash
kubectl get serviceaccount -n kube-system ebs-csi-controller-sa
```
Next, use the kubectl annotate command to associate the IAM role ARN with this service account. You can find the role's ARN in the AWS Console.
```bash
kubectl annotate serviceaccount ebs-csi-controller-sa -n kube-system eks.amazonaws.com/role-arn=arn:aws:iam::123456789012:role/EBS-CSI-Driver-Role
```
(Replace the ARN with the one for the role you just created.)

To ensure the changes take effect, you may need to restart the EBS CSI driver's controller pods.
```bash
kubectl rollout restart deployment -n kube-system ebs-csi-controller
```
Now, any pods that use the ebs-csi-controller-sa service account will automatically have the permissions from the associated IAM role.


## To Create S3 as a Volume to Attach with Kubernetes

## Step 1: Create S3 Bucket & Create the IAM Policy for S3

Firstly, you'll create S3 bucket and then because it is static persistent volume you need to create pv.yml like:
```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-s3-pv
spec:
  # The storage capacity is ignored by the S3 CSI driver, but it's
  # required by the CSI spec. A nominal value is sufficient.
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  # The Access Mode must be ReadWriteMany, as S3 is shared storage.
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: s3-sc
  csi:
    driver: s3.csi.aws.com
    volumeHandle: my-s3-pv-handle  # This must be unique per PV
    volumeAttributes:
      bucketName: "my-aws-s3-bucket-1575" # Replace with your bucket name
      # Optional: specify the AWS region if different from the cluster
      region: "us-east-1"
```

Then, you'll create a policy that defines the specific S3 permissions your pod needs. It's best practice to follow the principle of least privilege and only grant the permissions required for the task.

- Open the AWS Management Console and navigate to IAM.
- In the left-hand menu, click Policies, then Create policy.
- Switch to the JSON tab and paste the following policy document. This example grants permissions for read and write access to a specific bucket and all objects within it. You must replace <YOUR_BUCKET_NAME> with the name of your S3 bucket.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::<YOUR_BUCKET_NAME>",
                "arn:aws:s3:::<YOUR_BUCKET_NAME>/*"
            ]
        }
    ]
}
```
(Replace arn in resources with your bucket name)
- Click Next: Tags, then Next: Review.
- Give the policy a descriptive name (e.g., MyS3AccessPolicy) and click Create policy.

## Step 2: Create the IAM Role

Next, you will create an IAM role with a Web identity trust relationship. This is the part that links the role to your EKS cluster's OIDC provider.
- In the IAM Console, click Roles, then Create role.
- Select Web identity as the Trusted entity type.
- From the dropdowns, choose the following:
   - Identity provider: Your EKS cluster's OIDC URL (oidc.eks.us-east-1.amazonaws.com/id/6BB4C067DC5EBE722ED268A6DD3D8632).
   - Audience: sts.amazonaws.com.
- Click Next.
- On the Add permissions page, search for the policy you created earlier (MyS3AccessPolicy).
- Select the checkbox next to your policy and click Next.
- Give the role a name (e.g., S3AccessRole).
- Review the role's trust policy to make sure it looks correct and then click Create role.

## Step 3: Associate the Role with a Kubernetes Service Account

This is the final and most critical step. You'll associate the IAM role with a specific Kubernetes service account. This is the account your pod will use to inherit the S3 permissions.
- Create a Kubernetes service account in your cluster with the kubectl command. You can give it any name you want (e.g., s3-access-sa). It's a good idea to create a dedicated service account for this purpose.

```bash
kubectl create serviceaccount s3-access-sa -n <YOUR_NAMESPACE>
```
- Find the ARN of the IAM role you just created in the AWS Console.
- Annotate the service account with the ARN of the IAM role. This is the magic step that enables IRSA.
```bash
kubectl annotate serviceaccount s3-access-sa -n <YOUR_NAMESPACE> eks.amazonaws.com/role-arn=arn:aws:iam::<YOUR_ACCOUNT_ID>:role/S3AccessRole
```
- Now, when you create a pod that needs S3 access, you just have to specify the serviceAccountName in your pod's manifest.
```yml
apiVersion: v1
kind: Pod
metadata:
  name: my-s3-app
spec:
  serviceAccountName: s3-access-sa
  containers:
  - name: my-container
    image: my-image-that-needs-s3-access
```
Any pod that uses the s3-access-sa service account will now automatically get temporary credentials from AWS STS, allowing it to perform the S3 actions defined in your policy.

## Step 4: Install Mountpoint for Amazon S3 CSI Driver

To use S3 as a volume with Kubernetes we have to install drivers in cluster. To install it follow these steps:
- Go to your cluster & Click on Add-ons Tab
- Click on Get more add-ons on the top-right corner of Tab
- Select Mountpoint for Amazon S3 CSI Driver.
- Select IRSA role created in Step 2 (S3AccessRole)
- Click create.

## Step 5: Check Add-ons Created or Not

To check add-ons created or not use this command:
```bash
aws eks describe-addon --cluster-name <your-cluster-name> --region <your-region> --addon-name aws-mountpoint-s3-csi-driver
```
(Replace your cluster-name & region)
