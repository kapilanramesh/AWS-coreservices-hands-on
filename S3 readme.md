 Amazon S3 Bucket Explanation

### 1️⃣ **Create S3 Bucket**

- Go to **S3 Console → Create bucket**
    
- Give a **unique bucket name** like `my-first-s3-bucket-2025-1`
    
- Choose AWS Region (same as your EC2 if you want quicker access)
    
- **Uncheck "Block all public access"** only if you want the files to be publicly accessible (e.g., for static websites or image hosting)
    

> ✅ Recommended: Keep it **blocked** unless you absolutely need public access.

---

### 2️⃣ **Enable Bucket Versioning (Optional)**

- While creating or after bucket creation, go to:  
    **Properties → Bucket Versioning → Enable**
    
- Helps maintain **previous versions** of files you overwrite or delete.
    

> 🔁 This is **not the same as Linux `ln` (link)**. This is **actual object versioning** stored by AWS, not symbolic references.

---

### 3️⃣ **Upload Files**

- Go to your bucket → Click **Upload**
    
- Add files or folders → Upload
    

---

### 4️⃣ **Access or Download Files**

From an EC2 instance that has S3 permissions (like an IAM Role with `AmazonS3ReadOnlyAccess` or more):

bash

CopyEdit

`aws s3 cp "s3://my-first-s3-bucket-2025-1/scripts/file-name.pdf" .`

- Enclose in **quotes** if the file name has spaces.
    
- Downloads to current working directory (`.`).
    

---

### 5️⃣ **Make File Public (Optional)**

- Go to the object → Click **Actions → Make public using ACL**
    
- If ACL is **disabled** (locked), go to:
    
    - Bucket → **Permissions → Object Ownership** → Enable ACLs
        

> ⚠️ Not recommended for sensitive files. Use **pre-signed URLs** for secure sharing.

---

### ✅ You're Done!

You now know how to:

- Create and configure a bucket
    
- Upload/download files
    
- Handle versioning and access
    
- Use the AWS CLI for practical work
