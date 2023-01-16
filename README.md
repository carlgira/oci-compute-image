# OCI Generative AI Image
Terraform script to start a **stable-diffusion, bloom and dreambooth** in compute instance using a nvidia GPU in OCI.

**Stable Diffusion** is a state of the art text-to-image model that generates images from text.

**Bloom** is a open-science, open-access multilingual large language model (LLM), with 176 billion parameters, and was trained using the NVIDIA AI platform, with text generation in 46 languages

**Dreambooth** allow to fine-tune a stable diffusion model with your own data.

## Requirements

- Terraform
- ssh-keygen
- Huggingface account

## Configuration

1. Follow the instructions to add the authentication to your tenant https://medium.com/@carlgira/install-oci-cli-and-configure-a-default-profile-802cc61abd4f.

2. Clone this repository
```
# Stable diffusion 1.5
git clone https://github.com/carlgira/oci-compute-image.git
```

3. Set three variables in your path. 
- The tenancy OCID, 
- The comparment OCID where the instance will be created.
- The "Region Identifier" of region of your tenancy. https://docs.oracle.com/en-us/iaas/Content/General/Concepts/regions.htm
- OCID of the image to 

```
export TF_VAR_tenancy_ocid='<tenancy-ocid>'
export TF_VAR_compartment_ocid='<comparment-ocid>'
export TF_VAR_region='<home-region>'
export TF_VAR_image='<image-ocid>'
```

4. Execute the script generate-keys.sh to generate private key to access the instance
```
sh generate-keys.sh
```

## Build
To build simply execute the next commands. 
```
terraform init
terraform plan
terraform apply
```

**After applying, the service will be ready in about 10 minutes** (it will install OS dependencies, nvidia drivers, and start  stable-diffusion-web-ui, bloom-web-ui and dreambooth-webui.

## Clean
To delete the instance execute.
```
terraform destroy
```

## Troubleshooting
1. If one is the three apps (stable-diffusion-webui, bloom-webui, dreambooth-webui) is down, you can check the logs and the state of each service, with the commands.

```
systemctl status stabble-diffusion
systemctl status dreambooth
systemctl status bloom-webui
```

You can try to start the service by.
```
sudo systemctl start <service-name>
```

2. Once the training has started in dreambooth, you can check that is really working by running.
```
ps -ef | grep acc
```
It should appear 3 processes using the "accelerate" binary.

3. Error ***Error: 404-NotAuthorizedOrNotFound, shape VM.GPU2.1 not found***.
This could be happening because in your availability domain (AD) there is no a VM.GPU2.1 shape available. The script use by default the first AD, but maybe you have to change this manually.

Get the list of AD of your tenancy
```
oci iam availability-domain list
```

In the main.tf file, change the index number from "0" to other of the ADs of your region. (in the case that your region has more than one AD)
```
availability_domain = data.oci_identity_availability_domains.ADs.availability_domains[0].name
```
This error can also happen if in your region there is no VM.GPU2.1, in that case you have to change the region var before executing the scripts. 
```
export TF_VAR_region='<other-region>'
```

## References
- https://github.com/carlgira/oci-generative-ai
