# Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL

## Objectives

In this lab, you learn how to perform the following tasks:

  - Create a Cloud Storage bucket and place an image into it.

  - Create a Cloud SQL instance and configure it.

  - Connect to the Cloud SQL instance from a web server.

  - Use the image in the Cloud Storage bucket on a web page.

### Steps

1. Create a Cloud Storage bucket and place an image into it.

    1. Deploy a web server VM instance and include the startup script.

          ```
          gcloud compute instances create bloghost --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --metadata=startup-script=apt-get\ update$'\n'apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart --maintenance-policy=MIGRATE --tags=http-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=bloghost --reservation-affinity=any
          ```

    2. Use the following firewall rule.

          ```
          gcloud compute firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server
          ```

    3. Enter your chosen location into an environment variable called LOCATION.

          `export LOCATION=US`

    4. Make bucket named after your project ID.

          `gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID`

    5. Retrieve a banner image from a publicly accessible Cloud Storage location.

          `gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png`

    6. Copy the banner image to your newly created Cloud Storage bucket:

          `gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png`

    7. Modify the Access Control List of the object you just created so that it is readable by everyone.

          `gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png`

2. Create a Cloud SQL instance and configure it.

    1. Show the list of potential machine types.

          `gcloud sql tiers list`

    2. Create the instance.

          `gcloud sql instances create blog-db --tier=db-n1-standard-1 --region=us-central1-a`

    3. Set the password.

          `gcloud sql users set-password root --host=% --instance [INSTANCE_NAME] --password [PASSWORD]`

    4. Copy the Public IP address for your SQL instance to a text editor for use later in this lab.

          `gcloud compute instances list`

    5. Add user account. Set the username and password.

          `gcloud sql users create [USER_NAME] \
          --host=[HOST] --instance=[INSTANCE_NAME] --password=[PASSWORD]`

    6. Add network with name "web front end" and network as "external IP address/32".

          `gcloud sql instances patch [INSTANCE_ID] --authorized-networks=[EXTERNAL IP ADDRESS]/32...`

3. Configure an application in a Compute Engine instance to use Cloud SQL

    1. Connect to VM instance.

          `gcloud compute ssh bloghost`

    2. In your ssh session on bloghost, change your working directory to the document root of the web server.

          `cd /var/www/html`

    3. Use the nano text editor to edit a file called index.php.

          `sudo nano index.php`

  4. Paste the content below into the file:

                <html>
                <head><title>Welcome to my excellent blog</title></head>
                <body>
                <h1>Welcome to my excellent blog</h1>
                <?php
                $dbserver = "CLOUDSQLIP";
                $dbuser = "blogdbuser";
                $dbpassword = "DBPASSWORD";
                // In a production blog, we would not store the MySQL
                // password in the document root. Instead, we would store it in a
                // configuration file elsewhere on the web server VM instance.

                $conn = new mysqli($dbserver, $dbuser, $dbpassword);

                if (mysqli_connect_error()) {
                  echo ("Database connection failed: " . mysqli_connect_error());
                  } else {
                    echo ("Database connection succeeded.");
                  }
                  ?>
                  </body></html>           

  5. Press `Ctrl+O`, and then press Enter to save your edited file.

  6. Press `Ctrl+X` to exit the nano text editor.

  7. Restart the web server:

            sudo service apache2 restart

  8. Open a new web browser tab and paste into the address bar your `bloghost` VM instance's external IP address followed by `/index.php`.

            Result: Database connection failed: ...

  9. In your ssh session on bloghost. Use the nano text editor to edit index.php again.

            sudo nano index.php

  10. Replace `CLOUDSQLIP` with the Cloud SQL instance Public IP address that you noted above and replace `DBPASSWORD` with the Cloud SQL database password that you defined above.

  11. Save and exit.

  12. Restart the web server:

        `sudo service apache2 restart`

  13. Reload bloghost VM instance's external IP address web page.

              Result: Database connection succeeded.


4. Configure an application in a Compute Engine instance to use a Cloud Storage object.

      1. Copy the URL of the object called `my-excellent-blog.png` by replacing `PROJECT_ID` with your project ID:

              https://storage.googleapis.com/PROJECT_ID/my-excellent-blog.png

      2. Return to ssh session on your bloghost VM instance and enter this command:

              cd /var/www/html

      3. Use the nano text editor to edit index.php:

              sudo nano index.php

      4. Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the URL you copied earlier into the line.

      5. Paste this HTML markup immediately before the URL:

              <img src='

      6. Place a closing single quotation mark and a closing angle bracket at the end of the URL:

              '>

      7. Save and exit.

      8. Restart the web server:

              sudo service apache2 restart

      9. Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, its content now includes a banner image.
