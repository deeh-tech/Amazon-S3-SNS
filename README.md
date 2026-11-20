# Secure S3 File Sharing & Email Alerts

This project sets up a private cloud folder on **Amazon S3** where outside media partners can safely upload, view, and delete files, while admins receive automatic email alerts whenever files are added or removed.

## How It Works
1. **Cloud Storage (`Amazon S3`):** A private bucket named `cafe-danbucket22` in the `us-west-2` region stores shared media inside a dedicated `images/` folder.
2. **Restricted User Access (`AWS IAM`):**
   - Outside partners sign in using the `mediacouser` account, which belongs to the `mediaco` user group.
   - A custom IAM policy allows `mediacouser` to only upload (`PutObject`), download/view (`GetObject`), and delete (`DeleteObject`) files inside `cafe-danbucket22/images/*`.
   - Any attempt to change bucket settings, edit permissions, or access other folders is blocked.
3. **Instant Email Alerts (`Amazon SNS`):**
   - An SNS topic named `s3NotificationTopic` is connected to the `images/` folder.
   - Whenever a file is created (`s3:ObjectCreated:*`) or deleted (`s3:ObjectRemoved:*`), S3 triggers `s3NotificationTopic` to send an instant email alert to subscribed admins.

## Files in This Repository
- **`file.sh`** – Shell script with AWS CLI (`aws s3`) commands used to create the `cafe-danbucket22` bucket and upload initial files into the `images/` folder.
- **`file.json`** – JSON configuration file applied via `aws s3api` to link S3 upload/delete events on the `images/` prefix to `s3NotificationTopic`.

## Setup Steps
1. **Create the Bucket & Upload Files:** Run the commands in `file.sh` to create `cafe-danbucket22` in `us-west-2` and sync local media files to `s3://cafe-danbucket22/images/`.
2. **Set Up IAM Permissions:** Create the `mediaco` group and `mediacouser` account, then attach an IAM JSON policy granting `GetObject`, `PutObject`, and `DeleteObject` only on `arn:aws:s3:::cafe-danbucket22/images/*`.
3. **Create the SNS Topic & Email Subscription:** Create `s3NotificationTopic`, allow `s3.amazonaws.com` to publish messages to it, and confirm an admin email subscription.
4. **Enable S3 Event Notifications:** Apply `file.json` to `cafe-danbucket22` using the AWS CLI (`aws s3api put-bucket-notification-configuration`) so object changes inside `images/` trigger email alerts.

## Testing & Results
- **Allowed Actions Worked:** Confirmed `mediacouser` could upload, read, and delete files inside `cafe-danbucket22/images/`.
- **Unauthorized Actions Blocked:** Confirmed that trying to change file permissions (`put-object-acl`) or bucket policies returned an `AccessDenied` error.
- **Email Alerts Delivered:** Confirmed that uploading or removing a file in `images/` immediately sent an email notification through `s3NotificationTopic`.
