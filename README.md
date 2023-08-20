# CounterAct deployment on GCP

It is possible to deploy Forescout VM on GCP. There is probably more than one way to achieve it, but in this howto we will explore converting our VMWare Virtual Image as starting poiting to convert into a GCP VM. The process is not straight forward into the documentation so my intention is to document what need to be done in order to deploy it successfully.

Things to know before starting:
- If you are not doing it in an existing GCP account you will need to setup one. Keep in mind all the resources you create and run will be charged into you Credit Card, specially don't forget the VM running if not needed and you can remove the External IP from the NIC configuration. 
- I did this conversion into my personal account, so a lot of permissions were requested to be granted. This howto will not cover these permissions.
- I had to switch between regions to make sure resources were avaialable. This will specially happen if you use the console. The Web GUI will make it easy for you.
- I just cover here basic networking, in production you will need to make sure customer attach the right network communication in order to communicate with the onPremise site (VPN, Dedicated Link) 

1. First of all you need to create a bucket into the Cloud Storage and upload the VM package from our Downloads page. 

<img width="808" alt="image" src="https://github.com/ukstin/forescout-gcp/assets/23662994/e771e69e-41af-4814-a9b4-9d9fb0e271c2">

2. Go to Compute Engine and create a new disk. 
	Name: Define a name for the disk
	Source: Virtual Disk (VMDK, VHD)
	Virtual Disk File: search for the VMDK file you imported in your bucket on the first step
	Operating System on virtual disk: CentOS 7
	I do not tried to install guest packages because Forescout image do not have the default package manager into the CentOS. So it will generate an error if you try.
	This process take a long time to finish, this is the time to get a cup of coffee.

<img width="800" alt="image" src="https://github.com/ukstin/forescout-gcp/assets/23662994/5782fb80-b0dd-4f1c-9314-01f40d327f0b">

Equivalent Command Line: 
>gcloud compute images import forescout-image --source-file=gs://uk-forescout/CounterACT-8.4.2-668.iso_V.vmdk --description="Forescout 8.4.2 image converted from VMWare VMDK" --no-guest-environment --os=centos-7

3. Then create a disk and associate with the image

<img width="802" alt="image" src="https://github.com/ukstin/forescout-gcp/assets/23662994/ff0458a3-2a9d-4372-8113-419d2ede290c">

Equivalent command line:
>gcloud compute disks create forescout-842-disk --project=uk-forescout --type=pd-balanced --description=Forescout\ 8.4.2\ Disk --size=200GB --zone=us-central1-c --image=forescout-image --image-project=uk-forescout

4. Finally create a VM instance, the important thing here is to attach the previous created disk as the boot disk

<img width="1104" alt="image" src="https://github.com/ukstin/forescout-gcp/assets/23662994/aa4df700-a777-42fa-9efe-8673b651508f">
<img width="802" alt="image" src="https://github.com/ukstin/forescout-gcp/assets/23662994/7df1c394-fbcf-4183-ad02-b7a7c8f344c5">

Equivalent command line:
>gcloud compute instances create forescout-842-vm --project=uk-forescout --zone=us-central1-c --machine-type=e2-medium --network-interface=network-tier=PREMIUM,stack-type=IPV4_ONLY,subnet=default --maintenance-policy=MIGRATE --provisioning-model=STANDARD --service-account=318388346623-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --disk=boot=yes,device-name=forescout-842-disk,mode=rw,name=forescout-842-disk --labels=goog-ec-src=vm_add-gcloud --reservation-affinity=any

Compute engine do not enable Serial Console by default, so you will need to enable it on the project or on the specific instance to finish de setup process:
>gcloud compute project-info add-metadata --metadata serial-port-enable=TRUE
or
>gcloud compute instances add-metadata instance-name --metadata serial-port-enable=TRUE

Then follow with normal setup, and it's done! Hope it helps. 

Special thanks to Howie Koh for tips and all the help.

Any comment please let me know.
	
	


