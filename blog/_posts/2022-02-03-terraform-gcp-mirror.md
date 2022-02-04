---
layout: page
title: Terraform Provider Mirror Setup on GCS
categories:
  - tech
  - cloud
tags:
  - terraform
  - tech
  - cloud
comments: true
published: true
---

This article covers the basic steps for creating a terraform provider mirror in a Google GCS bucket. For those of you who are not familiar with terraform providers can read more about it [here](https://www.terraform.io/registry/providers) at [www.terraform.io](https://www.terraform.io/). 

1. Setup an example tf file with just the providers. See [Provider Configuration](https://www.terraform.io/language/providers/configuration)
   ```
   | cat main.tf 
   provider "google" {
     project = "acme-app"
     region  = "us-central1"
   }
   ```
2. Test providers download
   * Try below to download provider from internet
     ```
     terraform init
     ```
   * Verify
     ```
	 | terraform providers

	 Providers required by configuration:
	 .
	 └── provider[registry.terraform.io/hashicorp/google]   
     ```
3. Now create a local mirror. The command below will copy all providers for the above module to this local directory mirror. See [command reference](https://www.terraform.io/cli/commands/providers/mirror)
   ```
    | mkdir mirror
	| terraform providers mirror mirror/
	- Mirroring hashicorp/google...
	  - Selected v4.9.0 with no constraints
	  - Downloading package for darwin_amd64...
	  - Package authenticated: signed by HashiCorp   
   ```
4. Try the local mirror 
   * setup
     ```
     # remove the lock file
     rm .terraform.lock.hcl 

     # setup enhanced logging
     export TF_LOG=TRACE
     ```
   * setup local configuration to use the above local mirror      
     ```
	 export TF_CLI_CONFIG_FILE="$(pwd)/.terraformrctemp"
	 cat <<EOF > $TF_CLI_CONFIG_FILE
	 provider_installation {
	   filesystem_mirror {
	     path    = "/Users/admin/Documents/docs/docs_misc/technical/code/github/terraform-tutorial/mirror/"
	     include = ["hashicorp/google"]
	   }
	 }
	 EOF     
     ```
   * try the local mirror now
     ```
     terraform init
     ```
5. Setup remote mirror in GCS
   * create a **public** bucket
   * copy the local mirror to the GCP bucket (assumes gcloud cli has been setup correctly with region/project/etc)
     ```
     gsutil cp -r ../terraform-tutorial/mirror/ gs://terraform-registry
     ```
     note: this will copy the mirror dir to the bucket (and it will be on the path)
   * Test that access works
     ```
     | curl https://storage.googleapis.com/terraform-registry/mirror/registry.terraform.io/hashicorp/google/index.json
	 {
	   "versions": {
	     "4.9.0": {}
	   }
	 }
     ```  
6. Try the remote mirror
   * Update the CFG file to use the remote bucket mirror
     ```
	   | cat .terraformrctemp 
	   provider_installation {
	       network_mirror {
	       url    = "https://storage.googleapis.com/terraform-registry/mirror/"
	       include = ["hashicorp/google"]
	     }
	   }
     ```
   
   * test
     ```
     | terraform init
     Initializing provider plugins...
     - Reusing previous version of hashicorp/google from the dependency lock file
     2022-02-01T21:24:34.872-0600 [DEBUG] Querying available versions of provider registry.terraform.io/hashicorp/google at network mirror https://storage.googleapis.com/terraform-registry/mirror/
     2022-02-01T21:24:34.873-0600 [DEBUG] GET https://storage.googleapis.com/terraform-registry/mirror/registry.terraform.io/hashicorp/google/index.json
     2022-02-01T21:24:34.876-0600 [TRACE] HTTP client GET request to https://storage.googleapis.com/terraform-registry/mirror/registry.terraform.io/hashicorp/google/index.json
     2022-02-01T21:24:35.118-0600 [TRACE] providercache.fillMetaCache: scanning directory .terraform/providers
     2022-02-01T21:24:35.118-0600 [TRACE] getproviders.SearchLocalDirectory: found registry.terraform.io/hashicorp/google v4.9.0 for darwin_amd64 at .terraform/providers/registry.terraform.io/hashicorp/google/4.9.0/darwin_amd64
     2022-02-01T21:24:35.118-0600 [TRACE] providercache.fillMetaCache: including .terraform/providers/registry.terraform.io/hashicorp/google/4.9.0/darwin_amd64 as a candidate package for registry.terraform.io/hashicorp/google 4.9.0
     - Using previously-installed hashicorp/google v4.9.0

     Terraform has been successfully initialized!

     You may now begin working with Terraform. Try running "terraform plan" to see
     any changes that are required for your infrastructure. All Terraform commands
     should now work.

     If you ever set or change modules or backend configuration for Terraform,
     rerun this command to reinitialize your working directory. If you forget, other
     commands will detect it and remind you to do so if necessary.   

     ```   
     Note that it downloaded from our GCS mirror
