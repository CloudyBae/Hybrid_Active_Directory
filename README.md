# Hybrid_Active_Directory
Use the Cloudformation nested templates above to create the needed resources. Once everything is provisioned, go to running EC2 instances and copy the public IPv4 DNS for the JumpBox instance. Open Microsoft Remote Desktop and connect into that DNS. Once remote desktop is loaded up, this will be the "on-prem" environment. 

From the on-prem env, open up Remote Desktop and use the public IPv4 of the EC2 instance named "Client". Once connected this will now simulate the "on-prem" environment

In the on-prem env, there will be a fileshare with a file inside that will be used to migrate into AWS. There is also a self managed Microsoft Active Directory.

In AWS, create a new AWS Managed Microsoft Active Directory. Now there are 2 different Microsoft Active Directories, 1 on-prem and 1 in AWS.

Create a new Jumpbox EC2 instance for the AWS side. Get the public IPv4 DNS name. With the remote desktop, create another PC and connect to the DNS of the new Jumpbox for AWS. Inside this remote desktop, install the remote server administration tools by running 'Install-WindowsFeature -IncludeAllSubFeature RSAT' in powershell. Once tools are installed, restart the remote desktop. After the restart, you can access the Active Directory tools, open active directory u sers and computers and here you can see the managed active directory that is delivered by Directory Service.

In on-prem env, check if kerberos pre-authentication is enabled. Create the conditional forwarder for the AWS Active Directory.

In AWS, edit the outbound rules for the security group attached to the Active Directory. Create an outbound rule that allows for all ipv4 traffic so the domain controllers on-prem can communicate with AWS AD. On the AWS jumpbox remote desktop, make sure kerberos pre-authentication is enabled.

On the on-prem remote desktop, set up a new two-way forest trust to AWS.  

In the AWS AD, add a new two-way forest trust relationship with the same IPs of the conditional forwarders we used for the on-prem env.

In the AWS Jumpbox remote desktop, add an on-prem admin user into the AWS Delated Administrators Group. Now you can log into the AWS Jumpbox remote desktop with an on-prem admin user. Test if it is working properly by connecting to the AWS Jumpbox with on-prem admin credentials and being able to use AD tools. 

Create a new AWS FSx for Windows File Server. You can test that the file share is accessible both on-prem and AWS by searching for the DNS of the File System in the on-prem env. If you see a share folder, it is working. You can migrate data in the original on-prem file share to the new AWS file share. 

Since both file shares are Windows compatible, you can configure the Distributed File System. Create a new namespace with a new folder that allows access from both on-prem and AWS. You can copy the data from on-prem into the folder in the namespace, then you can disable the folder target in on-prem which makes this namespace only direct towards the FSx file share. 

Create a new AWS WorkSpace and register the Active Directory. Create a new on-prem admin user in the on-prem env. Refresh the WorkSpace and you should be able to see the new user that was just created on-prem. Create a new Windows based WorkSpace. 

Download and install WorkSpace. Use the registration code you created to register. You can sign into WorkSpaces with the admin account that was created on-prem because there is a two-way forest trust. Edit the security group associated with FSx to allow inbound access from the security group associated with WorkSpaces. Now you can access the file share on WorkSpaces.