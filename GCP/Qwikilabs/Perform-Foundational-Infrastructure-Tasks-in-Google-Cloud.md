# Perform Foundational Infrastructure Tasks in Google Cloud

[toc]

## Cloud Storage: Qwik Start - Cloud Console

- Cloud Storage allows world-wide storage and retrieval of any amount of data at any time. You can use Cloud Storage for a range of scenarios including serving website content, storing data for archival and disaster recovery, or distributing large data objects to users via direct download.
- Create a Bucket
  - **Navigation menu** > **Cloud Storage** > **Browser** > **Create a bucket**
    - Name: Project ID
    - Region: us-east1(South Carolina)
    - Default sotrage class: Standard
    - Access control: Uniform

- Upload an object into the Bucket
  - **Click Bucket Name** > **Upload files**
  - **Click ...** > **Rename**

- Share an Object Publicly
  - **Permissions**> **Public Access** > **Remove Public Access Prevention** > **Confirm**
  - **Permissions**> **Members** > **ADD** > **allUsers** > **Cloud Storage** > **Storage Object Viewer** > **Save**
  - **Object** > **Public access** > **Copy URL**
- Create Folders
- Delete Folders
- Download

## Cloud Storage: Qwik Start - CLI/SDK

- Create a Bucket

  - **Navigation menu** > **Cloud Storage** > **Browser** > **Create a bucket**
    - Name: Project ID
    - MultiRegion: us
    - Default sotrage class: Standard
    - **Uncheck** *Enforce public access prevention on this bucket* checkbox
    - Access control: Fine-grained

- Upload an object into your bucket

  ```sh
  wget --output-document ada.jpg https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg
  gsutil cp ada.jpg gs://${YOUR_BUCKET_NAME}rm ada.jpg
  ```

- Download an object from your bucket

  ```sh
  gsutil cp -r gs://${YOUR_BUCKET_NAME}/ada.jpg .
  ```

- Copy an object to a folder in the bucket

  ```sh
  gsutil cp gs://${YOUR_BUCKET_NAME}/ada.jpg gs://${YOUR_BUCKET_NAME}/image-folder/
  ```

- List contents of a bucket or folder

  ```sh
  gsutil ls gs://${YOUR_BUCKET_NAME}
  gsutil ls -l gs://YOUR-BUCKET-NAME/ada.jpg
  ```

- Make your object publicly accessible

  ```sh
  gsutil acl ch -u AllUsers:R gs://${YOUR_BUCKET_NAME}/ada.jpg
  ```

- Remove public access

  ```sh
  gsutil acl ch -d AllUsers gs://${YOUR_BUCKET_NAME}/ada.jpg
  ```

- Delete objects

  ```sh
  gsutil rm gs://${YOUR_BUCKET_NAME}/ada.jpg
  ```

## Cloud IAM: Qwik Start

- Google Cloud's Identity and Access Management (IAM) service lets you create and manage permissions for Google Cloud resources.

- [Google Cloud roles documentation](https://cloud.google.com/iam/docs/understanding-roles#primitive_roles)

- The IAM console and project level roles(Username1)

  - **Navigation menu** > **IAM & Admin** > **IAM** > **ADD** > **Select a role**
  - You should see Browser, Editor, Owner, and Viewer roles. These four are known as *primitive roles* in Google Cloud.
  - Cancel

- Explore editor roles(Username 2)

  - **Navigation menu** > **IAM & Admin** > **IAM**
  - You should see Username 2 has the "Viewer" role granted to the project
  - And the **ADD** button at the top is grayed out

- Prepare a resource for access testing(Username1)

  - **Navigation menu** > **Cloud Storage** > **Browser** > **Create a bucket**
    - Name: Project ID
    - MultiRegion: us
  - Upload any txt file
  - Reanme to `sample.txt`

- Verify project viewer access(Username 2)

  - **Navigation menu** > **Cloud Storage** > **Browser**
  - Username 2 has the "Viewer" role prescribed which allows them read-only actions that do not affect state

- Remove project access(Username 1)

  - **Navigation menu** > **IAM & Admin** > **IAM**
  - Remove Project Viewer access for **Username 2**(Pencil icon > Trashcan icon), then click **Save**

- Verify that Username 2 has lost access(Username2)

- Add Storage permissions(Usernam1)

  - **Navigation menu** > **IAM & Admin** > **IAM** >**ADD**
     - New members: Username2
     - Select a role: **Cloud Storage** > **Storage Object Viewer**
     - Save

- Verify access(Username2)

  ```sh
  gsutil ls gs://${YOUR_BUCKET_NAME}
  ```
