# How to copy from AWS S3 to Azure Storage

Recently, I needed to find a way to copy the entire content of an S3 bucket that has over 160,000 files to Azure blob storage. You can always write your own app to accomplish this and there's a couple of examples available. But for my case, I was looking for a ready, command line option. Here's what I tried:

Let's assume I have the following:

- A bucket in S3 named `my-s3-bucket`
- A storage account in Azure named `my-azure-storage-account`
- A storage container in Azure named `my-azure-storage-container`
- A folder that I will use as local storage named `my-local-folder`
- Azure Storage Access Key named `my-azure-storage-access-key`

There might be updates to the apps I used, so here's what I tested at the time of writing this post:

- AWS CLI 1.11.63
- blobxfer 0.12.1
- AzureCopy 1.4.0.7
- AzCopy 5.2.0

## AWS CLI S3 Sync command & blobxfer

The best solution for me was using the [Sync](http://docs.aws.amazon.com/cli/latest/reference/s3/sync.html) command in AWS CLI to download from S3, and then upload to Azure storage using [blobxfer](https://github.com/Azure/blobxfer). These 2 provided me with the most "Rsync-like" experience.

NOTE: If you don't need an Rysnc-like experience, you might also use AWS CLI [cp](http://docs.aws.amazon.com/cli/latest/reference/s3/cp.html) command instead of AWS CLI Sync.

#### Pros:
- Both AWS CLI and blobxfer are multi-platform. It's very easy to script them and run as a Cron job or as a scheduled task for ongoing sync.
- Both AWS CLI and blobxfer keep track of changes using MD5. So you don't download/upload the entire content every time. Only the changed files are processed. More info on AWS CLI checksum validation [here](http://docs.aws.amazon.com/cli/latest/topic/s3-faq.html).
- MD5 calculation was fast. In my case it took ~5 mins to calculate the MD5 of 161,000 files with a DS3_v2 VM in Azure.
- There's tons of options and great documentation for both AWS CLI and blobxfer.

#### Cons:
- Not a one-step solution. So you're not directly copying from S3 to Azure storage. You download to a local folder from S3 and upload from that local folder to Azure storage. That also means you're limited to your local machine's resources (e.g. bandwidth).

#### Usage:

You'll need to make sure you have [configured](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html) AWS CLI and you have the storage access key or the sas key for Azure storage.

First let's download from S3 using AWS CLI. Notice that I used the `--delete` parameter to make sure if there's any files that no longer exists in the source (`my-s3-bucket`) are also deleted from the target (`my-local-folder`).

`aws s3 sync s3://my-s3-bucket my-local-folder --delete`

Then I'll upload this files to Azure storage using blobxfer. Blobxfer also has a `--delete` parameter, doing exactly the same thing with AWS CLI. The `--upload` parameter tell blobxfer to upload the files. If you don't add it, it will get the files *from* Azure storage (`my-azure-storage-account`) instead of uploading to it.

`blobxfer my-azure-storage-account my-azure-storage-container my-local-folder --upload --delete --storageaccountkey my-azure-storage-access-key`


## AzureCopy

[AzureCopy](https://github.com/kpfaulkner/azurecopy) was the first thing I tried. Unfortunately, it didn't work for me because of a bug (see Cons).

#### Pros:
- You can copy from S3 to Azure in a single step without using a local folder if you use the `-blobcopy` parameter.
- Currently it's Windows only but there's a multi-platform [Go version](https://github.com/kpfaulkner/azurecopy-go) in the works.

#### Cons:
- At the time of this post, there's a [bug](https://github.com/kpfaulkner/azurecopy/issues/10) that prevents you from using a S3 bucket name with a period in its name. And yes, the S3 bucket I was trying to copy had a period in its name :)
- I might be wrong but I don't think that it calculates the checksum.
- If I use the `-blobcopy` parameter, the directory structure is not created recursively. I might be missing something but at least it didn't work for me.
- Not really Rysnc-like.

#### Usage:

Download the latest [release](https://github.com/kpfaulkner/azurecopy/releases) and open the `azurecopy.exe.config` file to add you AWS & Azure authentication keys.

Then run the following command. Please note that this will use the local machine's resources, so it will be downloading from S3 and uploading to Azure. You can add the `-blobcopy` option to copy directly from S3 to Azure storage, however, in my case that did not created the folder structure recursively.

`azurecopy -i https://my-s3-bucket.s3.amazonaws.com/ -o https://my-azure-storage-account.blob.core.windows.net/my-azure-storage-container`

## AWS CLI S3 Sync command & AzCopy

If you're on Windows, you can use this path, too.

#### Pros:
- AWS CLI & [AzCopy](https://docs.microsoft.com/en-us/azure/storage/storage-use-azcopy) are the "official" products of Amazon & Microsoft. 
- Can be easily scripted to run as a scheduled task.

#### Cons:
- [AzCopy](https://docs.microsoft.com/en-us/azure/storage/storage-use-azcopy) is currently Windows only.
- Not a one-step solution. So you're not directly copying from S3 to Azure storage. You download to a local folder from S3 and upload from that local folder to Azure storage. That also means you're limited to your local machine's resources (e.g. bandwidth).
- Not really Rysnc-like.

#### Usage:

Since AWS CLI is multiplatform, the first step is the same:

`aws s3 sync s3://my-s3-bucket my-local-folder --delete`

Then use AzCopy to upload to Azure Storage. Notice the `/S` parameter. That uploads the contents of the specified directory to Azure storage recursively.

`AzCopy /Source:C:my-local-folder /Dest:https://my-azure-storage-account.blob.core.windows.net/my-azure-storage-container /DestKey:my-azure-storage-access-key /S`

