# remote-comm-yakkety
Simple Ubuntu 17.04 image with various packages installed for remote server management. Meant to be deployed on any cloud instance and populated with SSH keys and credentials to manage Kubernetes, `git` repos, etc.

## What's included
* `kubectl`
* `gcloud`
* `gcsfuse`

## Usage
* Run `gcloud init` to get started with Google Cloud

### Cloud Storage FUSE
**Note: Run these commands on the host**
* You'll need [Google Cloud SDK](https://cloud.google.com/sdk/docs/#install_the_latest_cloud_tools_version_cloudsdk_current_version "Google Cloud SDK Documentation"), [`gcsfuse`](https://github.com/GoogleCloudPlatform/gcsfuse/blob/master/docs/installing.md "gcsfuse/installing.md at master · GoogleCloudPlatform/gcsfuse") and [Docker Community Edition](https://store.docker.com/search?type=edition&offering=community "Docker Store")
  * Run the `tools-*.sh` script for your host's distribution, e.g:
  
  ```bash
  chmod +x tools-ubuntu-debian.sh
  ./tools-ubuntu-debian.sh
  ```
  
* Obtain Application Default Credentials for `gcsfuse`
  * `gcloud auth application-default login`
  * Or use a [JSON file](https://developers.google.com/identity/protocols/application-default-credentials#howtheywork "How the Application Default Credentials work")
* Mount a GCS Bucket:

  ```bash
  sudo mkdir /mnt/gcsbucket && chown $USER:$USER /mnt/gcsbucket
  sudo sed -i 's/^#user_allow_other/user_allow_other/g' /etc/fuse.conf
  gcsfuse -o allow_other name_of_your_GCS_bucket /mnt/gcsbucket
  ```

### Secrets and Credentials
* Create a `Secret` directory in you bucket and encrypt it with `ecryptfs`

  ```bash
  cd /mnt/gcsbucket
  mkdir -m 700 .secret
  mkdir -m 500 secret
  sudo mount -t ecryptfs .secret/ secret/
  ```
  
* Go through the setup steps and write down your chosen passphrase.
  * You can unmount and re-encrypt your secrets with `sudo umount /mnt/gcsbucket/secret/`.
  * Repeat the `sudo mount -t ecryptfs .secret/ secret/` to mount your secret directory anytime, going through the interactive prompts and answering the same way as when you initially set it up.
  
* Start a container with the bucket and secrets mounted:
  `docker run -it -v /mnt/gcsbucket:/root/gcsbucket chepurko/remote-comm-yakkety /bin/bash`

* **NOTE: The Dockerfile relies on your directories being named `gcsbucket`, so if you must use a different dir name then you'll have to modify the Dockerfile on your own as welll.**
