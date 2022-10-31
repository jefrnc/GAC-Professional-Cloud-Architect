# Automating Infrastructure on Google Cloud with Terraform: Challenge Lab

## Task 1 : Create the configuration files

- Create a terraform file and folder structure using Cloud Shell by running the following command

```shell
mkdir terraform
cd terraform
touch main.tf variables.tf
mkdir modules
cd modules
mkdir instances
cd instances
touch instances.tf outputs.tf variables.tf
cd ..
mkdir storage
cd storage
touch storage.tf outputs.tf variables.tf
cd ../../
```

- Add the following script to the __variables.tf__ file and replace <PROJECT ID> with your GCP PROJECT ID

```yaml
variable "region" {
 default = "us-central1"
}

variable "zone" {
 default = "us-central1-a"
}

variable "project_id" {
 default = "<PROJECT ID>"
}
```

- Add the following script to the __main.tf__ file

```yaml
 terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "3.55.0"
    }
  }
}

provider "google" {
  project     = var.project_id
  region      = var.region
  zone        = var.zone
}

module "instances" {
  source     = "./modules/instances"
}
```

Run <code>terraform init</code> in Cloud Shell in the terraform root folder we created earlier

## Task 2 : Import infrastructure

Please open the __modules/instances/instances.tf__ file and add the script below

```yaml
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-1"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-1"
  zone         = "us-central1-a"

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```

For the 1st __Instance__
Then open the menu navigation to __Compute Engine > VM Instances__ click __tf-instance-1__ copy Instance ID which is under the scroll a little. Then run the following command with Cloud Shell to import, replace __<INSTANCE-ID>__ with the Instance ID earlier

```shell
 terraform import module.instances.google_compute_instance.tf-instance-1 <INSTANCE-ID>
```

For the 2nd __Instance__
Next, open the menu navigation to __Compute Engine > VM Instances__ click __tf-instance-2__ copy Instance ID which is under the scroll a bit. Then run the following command with Cloud Shell to import, replace __<INSTANCE-ID>__ with the Instance ID earlier

```shell
 terraform import module.instances.google_compute_instance.tf-instance-2 <INSTANCE-ID>
```

Both examples have now been imported into your terraform configuration. You can now optionally run commands to update Terraform state. Type <code>yes</code> in the dialog after you run the command for the status change to be accepted.

```shell
 terraform plan
 terraform apply
```

## Task 3 : Configure a remote backend

Add the following script to the __modules/storage/storage.tf__ file

```yaml
 resource "google_storage_bucket" "storage-bucket" {
  name          = "<PROJECT ID>"
  location      = "US"
  force_destroy = true
  uniform_bucket_level_access = true
 }
```

Next, please add the following script into the __main.tf__ file and paste it at the bottom

```yaml
 module "storage" {
  source     = "./modules/storage"
 }
```

Next run the following command to initialize the module and create a storage bucket resource. Type <code>yes</code> in the dialog after you run the apply command to accept the status change.

```shell
 terraform init
 terraform apply
```

Next, please replace the code with the script block as follows then change <PROJECT ID> with your GCP PROJECT ID to define the bucket

```yaml
 terraform { 
  backend "gcs" { 
    bucket = "<FILL IN PROJECT ID>" 
 prefix = "terraform / state" 
  } 
  required_providers { 
    google = { 
      source = "hashicorp/google" 
      version = "3.55.0" 
    } 
  } 
}
```

Jalankan script berikut untuk inisialisasi remote backend. Ketik <yes> jika di minta

``` shell
 terraform init
```

## Task 4 : Modify and update infrastructure

Silahkan buka file yang berada pada __modules/instances/instance.tf__ lalu lakukan replace semua script dengan script berikut untuk modifikasi instance dan menambahkan instance baru

```yaml
 resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}

resource "google_compute_instance" "tf-instance-3" {
  name         = "tf-instance-3"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```

Run the following command to initialize and update the module in the instance. Type <code>yes</code> if prompted

```shell
terraform init
terraform apply
```

## Task 5 : Taint and destroy resources

Stain for __tf-instance-3__ by running command

```shell
terraform taint module.instances.google_compute_instance.tf-instance-3
```

Run the following command to make a status change

```shell
terraform init
terraform apply
```

Next remove the resource for __tf-instance-3__. With the following code block in the file __modules/instances/instance.tf__

```yaml
resource "google_compute_instance" "tf-instance-3" {
  name         = "tf-instance-3"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "default"
  }
}
```

Run the following command to make a status change

```shell
terraform apply
```

## Task 6 : Use a module from the Registry

Please add the following script snippet into the __main.tf__ file to create a VPC module. Paste at the bottom

```yaml
module "vpc" {
    source  = "terraform-google-modules/network/google"

    project_id   = var.project_id
    network_name = "terraform-vpc"
    routing_mode = "GLOBAL"

    subnets = [
        {
            subnet_name           = "subnet-01"
            subnet_ip             = "10.10.10.0/24"
            subnet_region         = "us-central1"
        },
        {
            subnet_name           = "subnet-02"
            subnet_ip             = "10.10.20.0/24"
            subnet_region         = "us-central1"
            subnet_private_access = "true"
            subnet_flow_logs      = "true"
            description           = "This subnet has a description"
        }
    ]
}
```

Run the following command to initialize and create a VPC module. When <code>yes</code> if prompted

```shell
terraform init
terraform apply
```

Next open the __modules/instances/instances.tf__ file and change all scripts with the script below

```yaml
resource "google_compute_instance" "tf-instance-1" {
  name         = "tf-instance-1"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "terraform-vpc"
    subnetwork = "subnet-01"
  }
}

resource "google_compute_instance" "tf-instance-2" {
  name         = "tf-instance-2"
  machine_type = "n1-standard-2"
  zone         = "us-central1-a"
  allow_stopping_for_update = true

  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-10"
    }
  }

  network_interface {
 network = "terraform-vpc"
    subnetwork = "subnet-02"
  }
}
```

Run the following command to initialize module and instance updates in the VPC. Type <code>yes</code> if prompted

```shell
terraform init
terraform apply
```

## Task 7 : Configure a firewall

Please add the following script snippet into the __main.tf.__ file to add a firewall and replace <PROJECT_ID> with your GCP PROJECT ID. Paste at the bottom

```yaml
resource "google_compute_firewall" "tf-firewall" {
  name    = "tf-firewall"
 network = "projects/<PROJECT_ID>/global/networks/terraform-vpc"

  allow {
    protocol = "tcp"
    ports    = ["80"]
  }

  source_tags = ["web"]
  source_ranges = ["0.0.0.0/0"]
}
```

Run the following command to initialize the module. Type <code>yes</code> if prompted

```shell
terraform init
terraform apply
```

Reference:

- <https://github.com/yudifaturohman/GCP-AutomatingInfrastructureOnGoogleCloudWithTerraformChallengeLab/blob/master/README.md>
- <https://github.com/KwokHing/GSP345-Automating-Infrastructure-on-Google-Cloud-with-Terraform-Challenge-Lab>
- <https://github.com/apichlinski/GCPTerraformChallenge-GSP345>
