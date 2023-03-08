# Hybrid Active Directory
Use the Cloudformation nested templates above to create the needed resources. Once everything is provisioned, go to running EC2 instances and copy the public IPv4 DNS for the JumpBox instance. Open Microsoft Remote Desktop and connect into that DNS. Once remote desktop is loaded up, this will be the "on-prem" environment. 
![Screenshot_1](https://user-images.githubusercontent.com/109190196/223593947-e20a01bb-1dd0-4b4b-9b03-35b482ff064e.jpg)

From the on-prem env, open up Remote Desktop and use the public IPv4 of the EC2 instance named "Client". Once connected this will now simulate the "on-prem" environment
![Screenshot_3](https://user-images.githubusercontent.com/109190196/223593996-003ebc94-0339-4912-a514-19432202ce8d.jpg)

In the on-prem env, there will be a fileshare with a file inside that will be used to migrate into AWS. There is also a self managed Microsoft Active Directory.
![Screenshot_4](https://user-images.githubusercontent.com/109190196/223594069-8ac1169a-3c56-495f-b172-7dda6d57d748.jpg)
![Screenshot_5](https://user-images.githubusercontent.com/109190196/223594144-d082f322-f807-420f-a2f0-6089ca209ba0.jpg)

In AWS, create a new AWS Managed Microsoft Active Directory. Now there are 2 different Microsoft Active Directories, 1 on-prem and 1 in AWS.
![Screenshot_6](https://user-images.githubusercontent.com/109190196/223594152-bcf7e315-44aa-4485-a05e-9a90c9feb693.jpg)

Create a new Jumpbox EC2 instance for the AWS side. Get the public IPv4 DNS name. With the remote desktop, create another PC and connect to the DNS of the new Jumpbox for AWS. Inside this remote desktop, install the remote server administration tools by running 'Install-WindowsFeature -IncludeAllSubFeature RSAT' in powershell. Once tools are installed, restart the remote desktop. After the restart, you can access the Active Directory tools, open active directory u sers and computers and here you can see the managed active directory that is delivered by Directory Service.
![Screenshot_7](https://user-images.githubusercontent.com/109190196/223594188-acf4d58a-fde5-486e-9469-17b6c59aa651.jpg)
![Screenshot_9](https://user-images.githubusercontent.com/109190196/223594278-784b9244-2951-4ef9-95d0-77df9e4aacd3.jpg)

In on-prem env, check if kerberos pre-authentication is enabled. Create the conditional forwarder for the AWS Active Directory.
![Screenshot_10](https://user-images.githubusercontent.com/109190196/223594344-2628ed61-3589-48ed-bcc6-ecc4a4362069.jpg)

In AWS, edit the outbound rules for the security group attached to the Active Directory. Create an outbound rule that allows for all ipv4 traffic so the domain controllers on-prem can communicate with AWS AD. On the AWS jumpbox remote desktop, make sure kerberos pre-authentication is enabled.
![Screenshot_11](https://user-images.githubusercontent.com/109190196/223594387-9da8195a-76ec-4bf3-a9f0-036f590536bc.jpg)

On the on-prem remote desktop, set up a new two-way forest trust to AWS.  
![Screenshot_12](https://user-images.githubusercontent.com/109190196/223594414-fe2ec188-e829-4613-a212-ddfeaa978ddf.jpg)

In the AWS AD, add a new two-way forest trust relationship with the same IPs of the conditional forwarders we used for the on-prem env.
![Screenshot_13](https://user-images.githubusercontent.com/109190196/223594439-78d7b03e-61dc-44be-832a-338a26a72d20.jpg)

In the AWS Jumpbox remote desktop, add an on-prem admin user into the AWS Delated Administrators Group. Now you can log into the AWS Jumpbox remote desktop with an on-prem admin user. Test if it is working properly by connecting to the AWS Jumpbox with on-prem admin credentials and being able to use AD tools. 
![Screenshot_14](https://user-images.githubusercontent.com/109190196/223594522-0f3bc420-efa5-450a-bace-4e6618aa85b6.jpg)
![Screenshot_15](https://user-images.githubusercontent.com/109190196/223594576-00befd9b-65fb-4a3e-babb-33bdce30f09b.jpg)
![Screenshot_16](https://user-images.githubusercontent.com/109190196/223594587-031e5457-06ee-432d-89ce-7469007a8b20.jpg)

Create a new AWS FSx for Windows File Server. You can test that the file share is accessible both on-prem and AWS by searching for the DNS of the File System in the on-prem env. If you see a share folder, it is working. You can migrate data in the original on-prem file share to the new AWS file share. 
![Screenshot_17](https://user-images.githubusercontent.com/109190196/223594611-8530cb69-4cbc-4b9d-bc85-cad54734f87f.jpg)
![Screenshot_18](https://user-images.githubusercontent.com/109190196/223594639-e9e27cab-42be-4762-9be9-598f0e273a1a.jpg)

Since both file shares are Windows compatible, you can configure the Distributed File System. Create a new namespace with a new folder that allows access from both on-prem and AWS. You can copy the data from on-prem into the folder in the namespace, then you can disable the folder target in on-prem which makes this namespace only direct towards the FSx file share. 
![Screenshot_20](https://user-images.githubusercontent.com/109190196/223594720-884ae387-3ddc-496f-b12d-6d12097b06d8.jpg)

Create a new AWS WorkSpace and register the Active Directory. Create a new on-prem admin user in the on-prem env. Refresh the WorkSpace and you should be able to see the new user that was just created on-prem. Create a new Windows based WorkSpace. 
![Screenshot_22](https://user-images.githubusercontent.com/109190196/223594853-977d7174-52cc-4873-b785-48820b63dae5.jpg)
![Screenshot_23](https://user-images.githubusercontent.com/109190196/223594871-2091ff4e-051d-450e-a4fd-a2953d59618a.jpg)

Download and install WorkSpace. Use the registration code you created to register. You can sign into WorkSpaces with the admin account that was created on-prem because there is a two-way forest trust. Edit the security group associated with FSx to allow inbound access from the security group associated with WorkSpaces. Now you can access the file share on WorkSpaces.
![Screenshot_25](https://user-images.githubusercontent.com/109190196/223594940-24a9edc4-ace2-4109-9594-abdf13a3bbe2.jpg)
![Screenshot_26](https://user-images.githubusercontent.com/109190196/223594947-f86f9ea1-d049-401d-9795-a70ad1e692a9.jpg)

