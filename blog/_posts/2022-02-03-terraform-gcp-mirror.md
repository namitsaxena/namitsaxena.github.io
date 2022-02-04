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
    2022-02-01T21:24:34.582-0600 [INFO]  Terraform version: 1.1.4
    2022-02-01T21:24:34.582-0600 [INFO]  Go runtime version: go1.17.2
    2022-02-01T21:24:34.583-0600 [INFO]  CLI args: []string{"terraform", "init"}
    2022-02-01T21:24:34.583-0600 [TRACE] Stdout is a terminal of width 239
    2022-02-01T21:24:34.583-0600 [TRACE] Stderr is a terminal of width 239
    2022-02-01T21:24:34.583-0600 [TRACE] Stdin is a terminal
    2022-02-01T21:24:34.583-0600 [DEBUG] Attempting to open CLI config file:      /Users/admin/Documents/docs/docs_misc/technical/code/github/terraform-tutorial/.terraformrctemp
    2022-02-01T21:24:34.583-0600 [INFO]  Loading CLI configuration from /Users/admin/Documents/docs/docs_misc/technical/code/github/terraform-tutorial/.terraformrctemp
    2022-02-01T21:24:34.585-0600 [DEBUG] Not reading CLI config directory because config location is overridden by environment variable
    2022-02-01T21:24:34.585-0600 [DEBUG] Explicit provider installation configuration is set
    2022-02-01T21:24:34.586-0600 [TRACE] Selected provider installation method cliconfig.ProviderInstallationNetworkMirror("https://storage.googleapis.com/terraform-registry/mirror/") with includes [registry.terraform.io/hashicorp/google] and excludes []
    2022-02-01T21:24:34.589-0600 [INFO]  CLI command args: []string{"init"}

    Initializing the backend...
    2022-02-01T21:24:34.615-0600 [TRACE] Meta.Backend: no config given or present on disk, so returning nil config
    2022-02-01T21:24:34.616-0600 [TRACE] Meta.Backend: backend has not previously been initialized in this working directory
    2022-02-01T21:24:34.616-0600 [DEBUG] New state was assigned lineage "b15fa731-e9b3-b883-1d20-b9eb962307b0"
    2022-02-01T21:24:34.617-0600 [TRACE] Meta.Backend: using default local state only (no backend configuration, and no existing initialized backend)
    2022-02-01T21:24:34.617-0600 [TRACE] Meta.Backend: instantiated backend of type <nil>
    2022-02-01T21:24:34.621-0600 [TRACE] providercache.fillMetaCache: scanning directory .terraform/providers
    2022-02-01T21:24:34.621-0600 [TRACE] getproviders.SearchLocalDirectory: found registry.terraform.io/hashicorp/google v4.9.0 for darwin_amd64 at .terraform/providers/registry.terraform.io/hashicorp/google/4.9.0/darwin_amd64
    2022-02-01T21:24:34.621-0600 [TRACE] providercache.fillMetaCache: including .terraform/providers/registry.terraform.io/hashicorp/google/4.9.0/darwin_amd64 as a candidate package for registry.terraform.io/hashicorp/google 4.9.0
    2022-02-01T21:24:34.867-0600 [DEBUG] checking for provisioner in "."
    2022-02-01T21:24:34.868-0600 [DEBUG] checking for provisioner in "/Users/admin/Documents/software/bin"
    2022-02-01T21:24:34.868-0600 [TRACE] Meta.Backend: backend <nil> does not support operations, so wrapping it in a local backend
    2022-02-01T21:24:34.869-0600 [TRACE] backend/local: state manager for workspace "default" will:
     - read initial snapshot from terraform.tfstate
     - write new snapshots to terraform.tfstate
     - create any backup at terraform.tfstate.backup
    2022-02-01T21:24:34.869-0600 [TRACE] statemgr.Filesystem: reading initial snapshot from terraform.tfstate
    2022-02-01T21:24:34.869-0600 [TRACE] statemgr.Filesystem: snapshot file has nil snapshot, but that's okay
    2022-02-01T21:24:34.869-0600 [TRACE] statemgr.Filesystem: read nil snapshot

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
