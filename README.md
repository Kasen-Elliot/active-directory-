<p align="center">
<img src="https://github.com/Kasen-Elliot/active-directory-/assets/127895952/bf768998-06c4-4f20-b116-0af12f4191e3" alt="Microsoft Active Directory Logo"/>
</p>

<h1 align = "center">Installing and Configuring Active Directory in Microsoft Azure</h1>
This lab demonstrates how to install and configure Active Directory using Azure. We will be using two VMs on Azure that are on the same virtual network. One VM will be installed with Active Directory and configured to be the <b>Domain Controller</b> and other VM will be used as a <b>client</b>. Then, we will configure the Active Directory to allow the Client to join the domain as well as creating user accounts using a Powershell script. 

<br />

<h2>Environments and Technologies Used</h2>
<ul>
  <li>Microsoft Azure (Virtual Machines/Compute)</li>
  <li>Remote Desktop</li>
  <li>Active Directory Domain Services</li>
  <li>Powershell</li>
  <li>(OPTIONAL) Notepad for writing down usernames and passwords for VMs</li>
</ul>

<br />

<h2>Operating Systems Used</h2>
<ul>
  <li>Windows Server 2022</li>
  <li>Windows 10 Pro (21H2)</li>
</ul>

<br />

<h2>Installation Steps</h2>

<h3>Setting up Resources in Azure</h3>

<p>
  <ul>
    <li>Our Client VM should be installed normally using the Windows 10 image (OS)</li>
    <ul>
    </ul>      
    <li>Our Domain Controller VM using Active Directory should be created using the image <b>Windows Server 2022 Datacenter: Azure Edition - x64 Gen2</b></li>
    <ul>
      <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/65a8d95d-68ff-47be-839d-f28a7fd415f8" height="80%" width="80%"/></li>
    </ul>
    <li>After the VMs are created, we'll set the Domain Controller's IP Address as <i>static</i> since having it dynamic will make them difficult for the VM to communicate with our client VM.</li>
    <li>Go to your Virtual Machines in Azure and go to <b>Networking</b> then go to the link listed next to <b>Network Interface</b>. Head to <b>IP Configurations</b> under <b>settings</b>, go to the ipconfig link to open up a window to toggle the IP configuration and allocation to <b>Static</b>.</li>
    <ul>
      <li>IP Configuration for the Domain VM</li>
      <li><img src="https://github.com/Kasen-Elliot/active-directory-/assets/127895952/aab682ab-1510-4e9e-9461-108435db029a" height="80%" width="80%"/></li>
    </ul>
  </ul>
</p>

<br />

<h3>Ensuring Connectivity</h3>

<p>
  <ul>
    <li>Logging in to the Client VM, open the Command Prompt and enter the command <b>ping [Domain Controller Private IP Address] -t</b> to endlessly send ping in order ensure reachability with the Domain Controller. Connection should time out after the first ping due to the Domain Controller's Firewall Settings.</li>
    <ul>
      <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/180eaae3-623c-4288-9d40-29208f77db2c" height = 80% width = 80% /></li>
    </ul>
    <li>Logging into the Domain Controller VM, go to the <b>Windows Defender Firewall with Advanced Security</b>. Head to the <b>Inbound Rules</b> and enable the rules under the protocol <b>ICMPv4</b>, specifically <i>Core Networking Diagnostics - ICMP Echo Request (ICMPv4-In)</i></li>
    <ul>
	<li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/b85f5e2d-0305-462b-b495-1f747e8295ee" height = 80% width = 80% /></li>
    </ul>
    <li>Head back to the Client VM, and we should now be seeing replies</li>
    <ul>
	<li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/c8c72ce4-cbac-4cc8-94e9-fe157f10b5ee" height = 80% width = 80% /></li>
    </ul>
  </ul>
</p>

<br />

<h3>Installing Active Directory on the Domain Controller</h3>

<p>
  <ul>
	  <li>In your Domain Controller VM, go to the Server Manager Dashboard and click on <b>Add Roles and Features</b>. Go through the installation process and upon getting to <b>Server Roles</b>, make sure to check the box for <b>Active Directory Domain Services</b></li>
	  <ul>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/6ee20482-83b4-4fa2-b1d5-41d1e2b6a399" height = 80% width = 80% /></li>
	  </ul>
	 <li>Once installed, we now have to promote the server into a domain controller. To do so, you may notice a <b>warning notification</b> on the top right where the flag icon is. Click on that flag and click <b>Promote this server to a domain controller</b>. Click on Add a new forest and specify a domain name. For this tutorial, we'll name the domain <b>mydomain.com</b>, specifiy the password, and proceed with the install. Noted, you will be automatically signed out, re-log in through Remote Desktop, and installation is fully completed!</li>
	  <ul>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/704249d1-0034-4814-a9ef-eb83da0784c6" height = 80% width = 80% /></li>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/0cdb336f-54d8-484b-8874-2caf1e04324c" height = 80% width = 80% /></li>
    	 </ul>
  </ul>
</p>

<br />

<h3>Important Log In Note</h3>

<p>
  <ul>
    <li>When logging back in to the domain controller VM through Remote Desktop Connection, it is important to log in with the <b>context of the domain.</b></li>
    <li>Type out the domain path and then the name of the user. For example: <b>mydomain.com\labuser.</b></li>  
  </ul>
</p>

<br />

<h2>Configuration Steps</h2>

<h3>Creating Organizational Units (OUs) and Users</h3>

<p>
  <ul>
    <li>OUs act like folders that hold information, privileges, and login access of users in the directory</li>
    <li>In the Server Manager Dashboard, go to the <b>Tools</b> tab to open the Active Directory Users and Computers console, right click on the domain (mydomain.com) and make two OUs, <b>_ADMIN</b> and <b>_EMPLOYEES</b>.</li>
	  <ul>
		  <li>These OUs names are needed for a later step were we create multiple accounts</li>
	  </ul>
    <li>In the _ADMIN OU, we'll create the user <b>Jane Doe</b> with the user name <b>jane_admin</b> and password of your creation</li>
    <ul>
	<li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/c92c488e-f5d2-4fd2-a2fe-f1c40c95063e" height = 80% width = 80% /></li>
    </ul>
    <li>We'll be granting Jane admin privileges. Using the <b>Security Group</b>, right click on the user and open their <b>Properties</b>. Click Member Of then Add to apply the appropraite security group.</li>
    <ul>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/5ed21220-0aa6-41b2-a730-dc8ce3fe72d3" height = 80% width = 80% /></li>
    </ul>
    <li>Now, the user Jane will be used to log in from here on, using the login username jane_admin.</li>
  </ul>
</p>

<br />

<h3>Joining the Client to the Domain</h3>

<p>
  <ul>
    <li>First, we need to configure the Domain Name System (DNS) server. Go to your Client VM in the Azure Portal and go to <b>Networking</b> then go to the link listed next to <b>Network Interface</b>. Head to <b>DNS Servers</b> under <b>settings</b>, and set the DNS Server to <b>Custom</b>. Then, enter the Domain Controller's private IP address and save the changes. Restart the client VM in order to ensure the DNS changes are saved.</li>
    <ul>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/d5dc468d-0dd6-4d2e-a59a-fe367f856234" height = 80% width = 80% /></li>
    </ul>
    <li>In the System menu of the client VM, click on Rename this PC (advanced) and Change.</li>
    <li>Enter the domain and necessary credentials in order to let the client join the domain (logging in as jane_admin). It is important to note that the login credentials have to be input within the context of the domain path (mydomain.com\jane_admin).</li>
    <ul>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/4797e969-30de-4638-b501-6eae52fd9104" height = 80% width = 80% /></li>
    </ul>
    <li>The client should now be part of the domain (A popup should appear welcoming you to the domain). On the domain controller, the client should now appear in Computers in the Active Directory Users and Computers panel.</li>
    <ul>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/82f5f5f1-d824-42fb-9f3d-18bf2b9d0584" height = 80% width = 80% /></li>
    </ul>  
  </ul>
</p>

<br />

<h3>Setup Remote Desktop for non-administrative users on Client VM</h3>

<p>
  <ul>
    <li>Before users in the domain can use the client computer, Remote Desktop has to be enabled for non-administrative users.</li>
    <li>While logged in as the administrator (jane_admin), open <b>System Properties</b>. Click on <b>Remote Desktop</b> and Select users that can remotely access this PC.</li>  
    <li>Allow Domain Users access to Remote Desktop. Non-administrative users can now log in to the Client.</li>
    <ul>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/9e9e2e96-3a3e-45c3-9553-b4d69d7c7867" height = 80% width = 80% /></li>
    </ul>
  </ul>
</p>

<br />

<h3>Creating Users and attempt to log into the Client VM with one of the users</h3>

<p>
  <ul>
    <li>In the Domain Controller VM logged in as jane_admin, open <b>Powershell ISE</b> as an administrator</li>
    <li>Using <a href = "https://github.com/joshmadakor1/AD_PS/blob/master/Generate-Names-Create-Users.ps1">this powershell script</a>, we will create thousands of randomly generated accounts all with the password "Password1"</li>
    <li>In Powershell ISE, create a new file and copy-and-paste the powershell script into the file and then run the script</li>
    <ul>
	    <li>All these users are generated and put into the _EMPLOYEES Organizational Unit in the Active Directory</li>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/89730bc5-b553-4634-a15d-73d0e447991c" height = 80% width = 80% /></li>
    </ul>
    <li>Head to the Active Directory Users and Computers console and select a random username and obtain their login information by going to <b>Properties</b> and in the <b>Account</b> tab</li>
    <ul>
	    <li>The username generate should appear as <b>[first name].[last name]</b>, in this image the user is selecting "falojo.kugori"</li>
	    <li><img src = "https://github.com/Kasen-Elliot/active-directory-/assets/127895952/896088c9-276f-4c14-8dff-bd800fc2e81b" height = 80% width = 80% /></li>
    </ul>
    <li>Attempt to log in the Client VM using the generate username you have selected (username being <b>mydomain.com\username</b>) and the password "Password1"</li>
  </ul>
</p>

<br />
