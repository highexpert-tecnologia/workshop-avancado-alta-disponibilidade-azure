# Workshop Avançado Alta disponibilidade no Azure

Realização: High Expert

Este é o repositório que contém o material e Labs dos projectos da Workshop Hands-on

Para realizar as atividades dos Laboratórios Hands-on e projectos Mão na massa estamos utilizando o Portal do Azure no idioma Inglês a fim de manter o padrão e não haver erros, e se necessário caso tenha alguma dificuldade no entendimento você pode abrir o [Google Tradutor](https://translate.google.com.br/) e traduzir para melhor entendimento.

### Hands-on Lab

## Lab 1- Deploy Landing Zone Virtual machines

   ![Screenshot of Landing Zone](/LANDING-ZONE.png)

## Lab 1.1 - Create Virtual Network Application (10 minutes)

1. In the Azure portal, search for and select **Virtual networks**, and, on the **Virtual networks** blade, click **+ Add**.

1. Create a virtual network with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource Group | (create new) **RG-HighExpert-Store-Prod** |
    | Name | **VNET-HighExpert-Store-Prod** |
    | Region | the name of any Azure region available in the subscription you will use in this lab |
    | IPv4 address space | **10.2.0.0/16** |
    | Subnet name | **snet-frontend** |
    | Subnet address range | **10.2.2.0/24** |
    | | |

1. Click **Add a subnet** and create a new subnet with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **snet-backend** |
    | Address range (CIDR block) | **10.2.3.0/24** |
    | Network security group | **None** |
    | Route table | **None** |
    | | |

    | Setting | Value |
    | --- | --- |
    | Name | **snet-appgw** |
    | Address range (CIDR block) | **10.2.10.0/24** |
    | Network security group | **None** |
    | Route table | **None** |
    | | |

    | Setting | Value |
    | --- | --- |
    | Name | **snet-vmss** |
    | Address range (CIDR block) | **10.2.100.0/22** |
    | Network security group | **None** |
    | Route table | **None** |
    | | |

1. Accept the defaults and on the the **Tags** blade, click **Create**.

    | Name | Value |
    | --- | --- |
    | **project** | **highexpertstore** |
    | **company** | **highexpert** |
      
1. Click **Review and Create**. Let validation occur, and hit **Create** again to submit your deployment.

1. Once the deployment completes browse for **Virtual Networks** in the portal search bar. Within **Virtual networks** blade, click on the newly created virtual network.

1. Explore properties to Virtual networks.


## Deploy Web server High Availability (HA) Virtual machines
## Lab 1.2 - Deploy Virtual machines (30 minutes)
### Create a Web server VM

1. In the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, click **+ Add**.

1. On the **Basics** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource group | **RG-HighExpert-Store-Prod** |
    | Virtual machine name | **VMWEB01** |
    | Region | select same region the Resouce group | 
    | Availability options | **No infrastructure redundancy required** |
    | Security type | **Standard** | 
    | Image | **Windows Server 2022 Datacenter - Gen 2** |
    | Azure Spot instance | **No** |
    | Size | **Standard_B2s or DS1_v2** |
    | Authentication type | Password |
    | Username | **highexpert** |
    | Password | **high@pass123** |
    | Public inbound ports | **RDP (3389) and HTTP (80)** |
     | | |

1. Click **Next: Disks >** and, on the **Disks** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | OS disk type | **Standard SSD** |
    | Enable Ultra Disk compatibility | **No** |
     | | |

1. Click **OK** and, back on the **Networking** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Virtual Network | **VNET-HighExpert-Store-Prod** |
    | Subnet | **sbnet-frontend** |
    | Public IP | **VMWEB01-PI** |
    | NIC network security group | **Basic** |
	| Inbound Ports | **RDP (3389) and HTTP (80)** |
    | Place this virtual machine behind an existing load balancing solution? | **No** |
    | | |

1. Click **Next: Management >** and, on the **Management** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Boot diagnostics | **Enable with managed storage account (recommended)** |
    | Enable auto-shutdown | **Off** | 
     | | |

1. Accept the defaults and on the the **Tags** blade, click **Create**.

    | Name | Value |
    | --- | --- |
    | **project** | **highexpertstore** |
    | **cost center** | **0021** |
     | | |

1. On the **Review + Create** blade, click **Create**.

1. Connect Virtual machine, in the session RDP on **VMWEB01**.

1. Install the Web Server, configure Application code and publish Web site,  in the virtual machine open **Windows PowerShell ISE**. You can copy and paste this command and **Execute**.

   ```powershell
   Install-WindowsFeature Web-Server -IncludeManagementTools
   Install-WindowsFeature Web-Asp-Net45, Web-Net-Ext45, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Static-Content
   Remove-IISSite -Name "Default Web Site" -Confirm:$false
   Invoke-WebRequest 'https://github.com/highexpert-tecnologia/imersao-arquitetoazure/raw/refs/heads/main/LabFiles/HighExpertStore_v1.zip' -OutFile C:\inetpub\wwwroot\HighExpertStore_v1.zip
   Expand-Archive C:\inetpub\wwwroot\HighExpertStore_v1.zip C:\inetpub\wwwroot\
   & $env:SystemRoot\System32\inetsrv\appcmd.exe add apppool /name:HighExpertStorePool /managedRuntimeVersion:v4.0 /managedPipelineMode:Integrated
   & $env:SystemRoot\System32\inetsrv\appcmd.exe set apppool "HighExpertStorePool" /enable32BitAppOnWin64:false
   & $env:SystemRoot\System32\inetsrv\appcmd.exe add site /name:"HighExpertStore" /bindings:"http/*:80:" /physicalPath:"C:\inetpub\wwwroot\HighExpertStore"
   & $env:SystemRoot\System32\inetsrv\appcmd.exe set app "HighExpertStore/" /applicationPool:"HighExpertStorePool"
   ```

1. On the Virtual machine, start Browser and navigate on **http://localhost**.

1. If executed successfully it will display the **High Expert Store application** and it is expected to encounter an error when connecting to the database that we will correct in the next activities.

   ![Screenshot of the application](/Images/IMG02.png)

### Create a new Web server VM

1. In the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, click **+ Add**.

1. On the **Basics** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource group | **RG-HighExpert-Store-Prod** |
    | Virtual machine name | **VMWEB02** |
    | Region | select same region the Resouce group | 
    | Availability options | **Yes** if available in the subscription you will use in this lab or **No infrastructure redundancy required** |
    | Availability zone | Diffent zone of the **VMWEB01** | 
    | Security type | **Standard** | 
    | Image | **Windows Server 2022 Datacenter - Gen 2** |
    | Azure Spot instance | **No** |
    | Size | **Standard_B2s or DS1_v2** |
    | Authentication type | Password |
    | Username | **highexpert** |
    | Password | **high@pass123** |
    | Public inbound ports | **None** |
     | | |

1. Click **Next: Disks >** and, on the **Disks** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | OS disk type | **Standard SSD** |
    | Enable Ultra Disk compatibility | **No** |
     | | |

1. Click **OK** and, back on the **Networking** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Virtual Network | **VNET-HighExpert-Store-Prod** |
    | Subnet | **sbnet-frontend** |
    | Public IP | **None** |
    | NIC network security group | **None** |
    | Place this virtual machine behind an existing load balancing solution? | **No** |
    | | |

1. Click **Next: Management >** and, on the **Management** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Boot diagnostics | **Enable with managed storage account (recommended)** |
    | Enable auto-shutdown | **Off** | 
     | | |

1. Accept the defaults and on the the **Tags** blade, click **Create**.

    | Name | Value |
    | --- | --- |
    | **project** | **highexpertstore** |
    | **company** | **highexpert** |
     | | |

1. On the **Review + Create** blade, click **Create**.

1. Connect Virtual machine, in the session Azure Bastion on **VMWEB02**.

1. Install the Web Server, configure Application code and publish Web site,  in the virtual machine open **Windows PowerShell ISE**. You can copy and paste this command and **Execute**.

   ```powershell
   Install-WindowsFeature Web-Server -IncludeManagementTools
   Install-WindowsFeature Web-Asp-Net45, Web-Net-Ext45, Web-ISAPI-Ext, Web-ISAPI-Filter, Web-Static-Content
   Remove-IISSite -Name "Default Web Site" -Confirm:$false
   Invoke-WebRequest 'https://github.com/highexpert-tecnologia/imersao-arquitetoazure/raw/refs/heads/main/LabFiles/HighExpertStore_v1.zip' -OutFile C:\inetpub\wwwroot\HighExpertStore_v1.zip
   Expand-Archive C:\inetpub\wwwroot\HighExpertStore_v1.zip C:\inetpub\wwwroot\
   & $env:SystemRoot\System32\inetsrv\appcmd.exe add apppool /name:HighExpertStorePool /managedRuntimeVersion:v4.0 /managedPipelineMode:Integrated
   & $env:SystemRoot\System32\inetsrv\appcmd.exe set apppool "HighExpertStorePool" /enable32BitAppOnWin64:false
   & $env:SystemRoot\System32\inetsrv\appcmd.exe add site /name:"HighExpertStore" /bindings:"http/*:80:" /physicalPath:"C:\inetpub\wwwroot\HighExpertStore"
   & $env:SystemRoot\System32\inetsrv\appcmd.exe set app "HighExpertStore/" /applicationPool:"HighExpertStorePool"
   ```

1. On the Virtual machine, start Browser and navigate on **http://localhost**.

### Create a Database VM

1. In the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade, click **+ Add**.

1. On the **Basics** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Subscription | the name of the Azure subscription you will be using in this lab |
    | Resource group | **RG-HighExpert-Store-Prod** |
    | Virtual machine name | **VMDB01** |
    | Region | select same region the Resouce group | 
    | Availability options | **No infrastructure redundancy required** |
    | Security type | **Standard** | 
    | Image | **Free SQL Server License: SQL Server 2022 Developer on Windows Server 2022** |
    | Azure Spot instance | **No** |
    | Size | **Standard_B2s or DS1_v2** |
    | Authentication type | Password |
    | Username | **highexpert** |
    | Password | **high@pass123** |
    | Public inbound ports | **None** |
     | | |

1. Click **Next: Disks >** and, on the **Disks** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | OS disk type | **Standard SSD** |
    | Enable Ultra Disk compatibility | **No** |
     | | |

1. Click **OK** and, back on the **Networking** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Virtual Network | **VNET-HighExpert-Store-Prod-EastUS2-001** |
    | Subnet | **sbnet-backend** |
    | Public IP | **None** |
    | NIC network security group | **Basic** |
    | Place this virtual machine behind an existing load balancing solution? | **No** |
     | | |

1. Click **Next: Management >** and, on the **Management** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Boot diagnostics | **Enable with managed storage account (recommended)** |
    | Enable auto-shutdown | **Off** |  
     | | | 

1. Click **Next: SQL Server >** and, on the **SQL Server** tab of the **Create a virtual machine** blade, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | SQL authentication | **Enable** |
    | Storage configuration | Click **Change configuration** |
    | Data storage | Change **Disk type** to **32 GB** |
    | Log storage | Change **Disk type** to **16 GB** |
    | TempDB storage | Change **Shared drive space** to **Use local SSD drive** or **Share the data drive for tempdb** |
     | | |
    
1. Accept the defaults and on the the **Tags** blade, click **Create**.

    | Name | Value |
    | --- | --- |
    | **project** | **highexpertstore** |
    | **cost center** | **0021** |
     | | |

1. On the **Review + Create** blade, click **Create**.

1. Connect Virtual machine, in the session RDP on **VMWEB01**.

1. On SQL Server the virtual machine, open by running the **SQL Server Management Studio** on Administrator.

1. On **SQL Server Management Studio**, select to **Trust server certificate** on Connection string in Connect SQL Server.  

1. Execute Script Database by running the **SQL Server Management Studio**, click **New Query**. You can copy and paste this command and **Execute**.

   ```powershell
   IF DB_ID('HighExpertStore') IS NULL BEGIN CREATE DATABASE HighExpertStore; END
   GO
   USE HighExpertStore;
   GO
   IF OBJECT_ID('dbo.Users','U') IS NULL
   CREATE TABLE dbo.Users (Id INT IDENTITY PRIMARY KEY, Name NVARCHAR(100) NOT NULL, Email NVARCHAR(200) NOT NULL UNIQUE, PasswordHash NVARCHAR(256) NOT NULL, PasswordSalt NVARCHAR(100) NOT NULL, CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME());
   GO
   IF OBJECT_ID('dbo.Sessions','U') IS NULL
   CREATE TABLE dbo.Sessions (Token UNIQUEIDENTIFIER PRIMARY KEY, UserId INT NOT NULL FOREIGN KEY REFERENCES dbo.Users(Id), CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(), ExpiresAt DATETIME2 NOT NULL);
   GO
   IF OBJECT_ID('dbo.Products','U') IS NULL
   CREATE TABLE dbo.Products (Id INT IDENTITY PRIMARY KEY, Name NVARCHAR(200) NOT NULL, Description NVARCHAR(MAX) NULL, Price DECIMAL(10,2) NOT NULL, Category NVARCHAR(100) NOT NULL, ImageUrl NVARCHAR(500) NULL, Active BIT NOT NULL DEFAULT 1, EstimatedWeightGrams INT NULL);
   GO
   IF OBJECT_ID('dbo.Carts','U') IS NULL
   CREATE TABLE dbo.Carts (Id INT IDENTITY PRIMARY KEY, UserId INT NOT NULL FOREIGN KEY REFERENCES dbo.Users(Id), Status NVARCHAR(20) NOT NULL DEFAULT 'Active');
   GO
   IF OBJECT_ID('dbo.CartItems','U') IS NULL
   CREATE TABLE dbo.CartItems (Id INT IDENTITY PRIMARY KEY, CartId INT NOT NULL FOREIGN KEY REFERENCES dbo.Carts(Id), ProductId INT NOT NULL FOREIGN KEY REFERENCES dbo.Products(Id), Quantity INT NOT NULL DEFAULT 1);
   GO
   IF OBJECT_ID('dbo.Orders','U') IS NULL
   CREATE TABLE dbo.Orders (Id INT IDENTITY PRIMARY KEY, UserId INT NOT NULL FOREIGN KEY REFERENCES dbo.Users(Id), OrderNumber NVARCHAR(50) NOT NULL UNIQUE, Subtotal DECIMAL(10,2) NULL, Discount DECIMAL(10,2) NULL, Shipping DECIMAL(10,2) NULL, Total DECIMAL(10,2) NOT NULL, Cep NVARCHAR(20) NULL, CouponCode NVARCHAR(50) NULL, Status NVARCHAR(20) NOT NULL, CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME());
   GO
   IF OBJECT_ID('dbo.OrderItems','U') IS NULL
   CREATE TABLE dbo.OrderItems (Id INT IDENTITY PRIMARY KEY, OrderId INT NOT NULL FOREIGN KEY REFERENCES dbo.Orders(Id), ProductId INT NOT NULL FOREIGN KEY REFERENCES dbo.Products(Id), Quantity INT NOT NULL DEFAULT 1, UnitPrice DECIMAL(10,2) NOT NULL);
   GO
   IF OBJECT_ID('dbo.Wishlist','U') IS NULL
   CREATE TABLE dbo.Wishlist (Id INT IDENTITY PRIMARY KEY, UserId INT NOT NULL FOREIGN KEY REFERENCES dbo.Users(Id), ProductId INT NOT NULL FOREIGN KEY REFERENCES dbo.Products(Id), CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(), CONSTRAINT UQ_Wishlist UNIQUE (UserId, ProductId));
   GO
   IF OBJECT_ID('dbo.Coupons','U') IS NULL
   CREATE TABLE dbo.Coupons (Id INT IDENTITY PRIMARY KEY, Code NVARCHAR(50) NOT NULL UNIQUE, Description NVARCHAR(200) NULL, DiscountType NVARCHAR(10) NOT NULL, DiscountValue DECIMAL(10,2) NOT NULL, Active BIT NOT NULL DEFAULT 1, ExpiresAt DATETIME2 NULL);
   GO
   IF NOT EXISTS (SELECT 1 FROM dbo.Coupons WHERE Code='BEMVINDO10') INSERT INTO dbo.Coupons(Code,Description,DiscountType,DiscountValue,Active) VALUES ('BEMVINDO10','10% de boas-vindas','percent',10,1);
   IF NOT EXISTS (SELECT 1 FROM dbo.Coupons WHERE Code='CLOUD20')  INSERT INTO dbo.Coupons(Code,Description,DiscountType,DiscountValue,Active) VALUES ('CLOUD20','R$20 off em qualquer compra','value',20,1);
   GO
   IF NOT EXISTS (SELECT 1 FROM dbo.Products)
   BEGIN
   INSERT INTO dbo.Products(Name,Description,Price,Category,ImageUrl,Active,EstimatedWeightGrams) VALUES
   (N'Camiseta High Expert - Azure Architect',N'Camiseta premium para arquitetos de nuvem',79.90,N'Camisetas',N'static/img/prod_camiseta_azure.png',1,250),
   (N'Camiseta High Expert - DevOps',N'Estilo e performance para quem vive CI/CD',79.90,N'Camisetas',N'static/img/prod_camiseta_devops.png',1,250),
   (N'Caneca High Expert - Cloud Lover',N'Caneca 350ml para começar o dia no Azure',49.90,N'Canecas',N'static/img/prod_caneca_cloud.png',1,350),
   (N'Adesivo High Expert - Kubernetes',N'Adesivo vinil recortado do K8s',14.90,N'Acessorios',N'static/img/prod_adesivo_k8s.png',1,20),
   (N'Caneca High Expert - MVP Edition',N'Edição especial para colecionadores',69.90,N'Canecas',N'static/img/prod_caneca_mvp.png',1,350);
   END
   GO
   IF COL_LENGTH('dbo.Orders','Address') IS NULL
   ALTER TABLE dbo.Orders ADD Address NVARCHAR(400) NULL;
   GO
   ```

1. The successful execution of the SQL Script will be displayed as shown in the following image.

   ![Screenshot of the application](/Images/IMG03.png)

1. Within the computer, start Browser and navigate on **Public IP Address** of the **VMWEB01**.

   ![Screenshot of the application](/Images/IMG04.png)

## Lab 2.1 - Configure Network Security groups (15 minutes)

1.  In the Azure portal, select **+ Create a resource**. In the **Search the Marketplace** box, **Network security group** and press Enter. Select it and on the **Network security group** blade, select **Create**.

2.  On the **Create network security group** blade, enter the following information, and select **Review + Create** then **Create**:
   
    -  Subscription: **Select your subscription**.

    -  Resource group: **RG-HighExpert-Store-Prod**

    -  Name: **NSG-HighExpert-Store-Prod_snet-backend**

    -  Region: This must match the location in which you created the **VNET-HighExpert-Store-Prod** virtual network.

3.  In the Azure Portal, navigate to **All Services**, type **Network security groups** the search box and select **Network security groups**.

4.  On the **Network security groups** blade, select **NSG-HighExpert-Store-Prod_snet-backend**. 

5.  On the **NSG-HighExpert-Store1_snet-backend** blade, select **Inbound security rules** under **Settings** on the left and select **Add**.

6.  On the **Add inbound security rule** blade, enter the following information, and select **Add**:

    -  Source: **IP Address**

    -  Source IP addresses/CIDR ranges: **10.2.2.0/24**

    -  Source port ranges: **(*)**

    -  Destination: **IP Address**

    -  Destination IP addresses/CIDR ranges: **10.2.3.0/24**

    -  Destination port ranges: **1433**

    -  Protocol: **TCP**

    -  Action: **Allow**

    -  Priority: **300**

    -  Name: **AllowDataTierInboundTCP1433**

7.  On the **NSG-HighExpert-Store-Prod_snet-backend - Inbound security rules** blade, select **Add**.

8.  On the **Add inbound security rule** blade, enter the following information, and select **Add**:

    -  Source: **Any**

    -  Source port ranges: **(*)**

    -  Destination: **Any**

    -  Destination port ranges: **(*)**

    -  Protocol: **Any**

    -  Action: **Deny**

    -  Priority: **4000**

    -  Name: **DenyAnyCustomAnyInbound**

15. On the **NSG-HighExpert-Store-Prod_snet_backend - Inbound security rules** blade, select **Subnets** under **Settings** and then select **+ Associate**.

16. On the **Associate subnet** blade, select **snet_backend** on the **Virtual network**.

17. Select **OK** at the bottom of the **Associate subnet** blade.

1. On the **Create network security group** blade, enter the following information, and select **Review + Create** then **Create**:
   
    -  Subscription: **Select your subscription**.

    -  Resource group: **RG-HighExpert-Store-Prod**

    -  Name: **NSG-HighExpert-Store-Prod_snet-frontend**

    -  Region: This must match the location in which you created the **VNET-HighExpert-Store-Prod** virtual network.

3.  In the Azure Portal, navigate to **All Services**, type **Network security groups** the search box and select **Network security groups**.

4.  On the **Network security groups** blade, select **NSG-HighExpert-Store-Prod_snet-frontend**. 

5.  On the **NSG-HighExpert-Store-Prod_snet-frontend** blade, select **Inbound security rules** under **Settings** on the left and select **Add**.

6.  On the **Add inbound security rule** blade, enter the following information, and select **Add**:

    -  Source: **Any**

    -  Source port ranges: **(*)**

    -  Destination: **IP Address**

    -  Destination IP addresses/CIDR ranges: **10.2.2.0/24**

    -  Destination port ranges: **80**

    -  Protocol: **TCP**

    -  Action: **Allow**

    -  Priority: **300**

    -  Name: **AllowWebTierInboundTCP80**

15. On the **NSG-HighExpert-Store-Prod_snet_frontend - Inbound security rules** blade, select **Subnets** under **Settings** and then select **+ Associate**.

16. On the **Associate subnet** blade, select **snet_frontend** on the **Virtual network**.

17. Select **OK** at the bottom of the **Associate subnet** blade.

1. In the **VMWEB01** Virtual machine, open **Windows Powershell** and run command the following to test connectivity to **VMDB01**.

   ```powershell
   Test-NetConnection -ComputerName VMDB01 -Port 1433
   ```

1. Verify that the output of the command includes the private IP address of **VMDB01**.

1. Examine the navegate on Application to Database was successful.

## Lab 2.2 - Deploy Azure Load Balancer (15 minutes)

1. In the Azure portal, search and select **Load balancers** and, on the **Load balancers** blade, click **+ Create**.

1. Create a **Standard Load balancer** with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Resource group | **RG-HighExpert-Store-Prod** |
    | Name | **ALB-HighExpert-Store** |
    | Region| name of the Azure region into which you deployed all other resources in this lab |
    | SKU | **Standard** |
    | Type | **Public** |
    | Tier | **Regional** |

1. On the Create Load balancer blade, select **Next : Frontend IP configuration >**.

1. Click **+ Add a Frontend IP configuration** with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Subscription | the name of the Azure subscription you are using in this lab |
    | Name | **ALB-HighExpert-Store** |
    | IP | **IPv4** |
    | Public IP address | **Create new** |
    | Public IP address name | **ALB-HighExpert-Store-PI** |
    | Availability zone | **Zone-redundant** |

1. Click **Backend pools** and click **+ Add a Backend pool**.

1. Add a Backend pool with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **BP-HighExpert-Store** |
    | Virtual network | **VNET-HighExpert-Store** |
    | IP version | **IPv4** |
    | Virtual machine | **VMWEB01** | 
    | Virtual machine IP address | associate IP address |
    | Virtual machine | **VMWEB02** |
    | Virtual machine IP address | associate IP address |

1. Click **Inboud rules**, and then click **+ Add a Load balancing rules**.

1. Add a load balancing rule with the following settings (leave others with their default values):

    | Setting | Value |
    | --- | --- |
    | Name | **LBR-HighExpert-Store** |
    | IP Version | **IPv4** |
    | Frontend IP address | **ALB-HighExpert-Store** | 
    | Backend pool | **BP-HighExpert-Store1** |
    | Protocol | **TCP** |
    | Port | **80** |
    | Backend port | **80** |
    | Health probe | **Create new** |
    | Health probe name | **HP-HighExpert-Store1** |
    | Session persistence | **None** |
    | Idle timeout (minutes) | **4** |
    | TCP reset | **Disabled** |
    | Floating IP (direct server return) | **Disabled** |
    | Outbound source network address translation (SNAT). | **Enabled (Recommended)** |

1. Accept the defaults and on the the **Tags** blade, click **Create**.

    | Name | Value |
    | --- | --- |
    | **project** | **highexpertstore** |
    | **company** | **highexpert** |
     | | |

1. On the **Review + Create** blade, click **Create**.

1. Wait for the load balancing rule to be created, click **Overview**, and note the value of the **IP address** of **Frontend IP configuration**.

1. Within the computer, start Browser and navigate on **Public IP Address** of the **ALB-HighExpert-Store**.

1. On the **VMWEB01** Virtual machine, start **Internet Information Services (IIS) Manager**, open **Sites** and select **HighExpertStore** in **Manage Website**, select **Stop** Web site.

1. Within the computer, start Browser and navigate on **Public IP Address** of the **ALB-HighExpert-Store** and validate the operation and redirection of the Azure Load balancer in action for the **VMWEB02** web server.

## Lab 2.3 - Deploy Azure Application Gateway (30 minutes)

1. In the Azure portal, search and select **Application gateways** and, on the **Application gateways** blade, click **+ Create**.

1. On the **Basics** tab, enter these values for the following application gateway settings:

   - **Resource group**: Select **RG-HighExpert-Store-Prod**
   - **Application gateway name**: Enter **AAG-WEB-Prod** for the name of the application gateway.
   - **Region**: select same region the Resouce group
   - **Tier**: Select **Standard v2**.
   - **Enable autoscaling**: Select **No**.
   - **Instance count**: Select **1**.

 2. Under **Configure virtual network**, select **VNET-HighExpert-Store-Prod**
    
3. On the **Basics** tab, accept the default values for the other settings and then select **Next: Frontends**.

1. On the **Frontends** tab, verify **Frontend IP address type** is set to **Public**.

2. Select **Add new** for the **Public IP address** and enter **AAG-WEB-Prod-PI** for the public IP address name, and then select **OK**. 

3. Select **Next: Backends**.

1. On the **Backends** tab, select **Add a backend pool**.

2. In the **Add a backend pool** window that opens, enter the following values to create an empty backend pool:

    - **Name**: Enter **BP-WEB** for the name of the backend pool.
    - **Add Target type**: Select **Virtual machine** and **Target** add **VMWEB01** and **VMWEB02**. 

3. In the **Add a backend pool** window, select **Add** to save the backend pool configuration and return to the **Backends** tab.

4. On the **Backends** tab, select **Next: Configuration**.

1. On the **Configuration** tab, you'll connect the frontend and backend pool you created using a routing rule.

1. Select **Add a routing rule** in the **Routing rules** column.

2. In the **Add a routing rule** window that opens, enter **RR-WEB** for the **Rule name**.

3. A routing rule requires a listener. On the **Listener** tab within the **Add a routing rule** window, enter the following values for the listener:

    - **Listener name**: Enter **L-WEB** for the name of the listener.
    - **Frontend IP**: Select **Public** to choose the public IP you created for the frontend.
  
      Accept the default values for the other settings on the **Listener** tab, then select the **Backend targets** tab to configure the rest of the routing rule.

4. On the **Backend targets** tab, select **BP-WEB** for the **Backend target**.

5. For the **HTTP setting**, select **Add new** to add a new HTTP setting. The HTTP setting will determine the behavior of the routing rule. In the **Add an HTTP setting** window that opens, enter **HTTPS-WEB** for the **HTTP setting name** and *80* for the **Backend port**. Accept the default values for the other settings in the **Add an HTTP setting** window, then select **Add** to return to the **Add a routing rule** window. 

6. On the **Add a routing rule** window, select **Add** to save the routing rule and return to the **Configuration** tab.

1. Accept the defaults and on the the **Tags** blade, click **Create**.

    | Name | Value |
    | --- | --- |
    | **project** | **highexpertstore** |
    | **company** | **highexpert** |
     | | |

7. Select **Next: Tags** and then **Next: Review + create**.

7. Wait for the deployment to complete before proceeding to the next step.

1. Test the Application gateway.

2. Find the public IP address for the Application gateway on its **Overview** page. 

2. Copy the public IP address, and then paste it into the address bar of your browser to browse that IP address.

3. Check the response. A valid response verifies that the application gateway was successfully created and can successfully connect with the backend.

1. Refresh the browser multiple times and you should see connections to both **VMWEB01** and **VMWEB02**.

## Lab 3 - Azure Virtual Machine Scale Sets (60 minutes)

   ![Screenshot of the inventory data](/Images/vmss-azure.png)

1. In the Azure portal, search for and select **Virtual machines** and, on the **Virtual machines** blade.

1. On the **Virtual machines** blade, find **VMWEB01** and **VMWEB02** and delete two VMS.

1. In the Azure portal, search for and select **Resource groups** and, on the **Resource groups** blade. 

1. On the **Resource groups** blade, find **RG-HighExpert-Hub** and delete Resource Group and all resource for this Resoure group.

1. In the Azure portal, search for **Virtual machine scale (VMSS)**. Select **Create** on the **Virtual Machine Scale Set (VMSS)** page, which will open the **+ Create** page.

1. In the **Basics** tab, under **Project details**, make sure the correct subscription is selected and then choose to **Create new** resource group. Type **RG-HighExpert-Store-VMSS** for the name and then select **OK**.

1. Type **vmsshestore** as the name for your scale set.

1. In **Region**, select a region that is close to your area.

1. In **Availability zone**, select a **None**.

1. In **Orchestration mode**, select **Flexible**.

1. In **Security type**, select **Standard**

1. In **Scaling mode**, select **Manually update the capacity** and **Instance count**, select **1**.

1. In **Image**, select **Windows Server 2022 Datacenter - x64 Gen2**.

1. In **Sizes**, select **Standard_B2s** or **DS1_v2** or **other size available**

1. In **Username** insert **highexpert** and **Password** insert **high@pass123**, and repeat **Password**

1. Select **Next** to move the the other pages. 

1. Leave the defaults for the **Spot** and **Disks** pages.

1. On the **Networking** page, under **Virtual network**, select **VNET-HighExpert-Store-Prod** and **Subnet** select **snet-vmss**.

1. On the **Networking** page, under **Network interface**, select **Edit network interface**, in **Name** insert **vmsshestore-nic01** and select **Subnet** **snet-vmss**, and click **OK**.

1.1. On the **Networking** page, under **Network interface**, in **Public inbound ports**, select Allow select ports **HTTP (80)** and **Public IP address** select **Disabled**.

1. On the **Networking** page, under **Load balancing**, select **Azure load balancer** to put the scale set instances behind a load balancer. 

1. In **Select a load balancer**, create a new **Azure Load Balancer**.

1. On **Load balancer name**, insert **vmsshestore-alb**, then select **Create** and click **Next**.

1. On the **Management** page, under **Upgrade mode**, select **Manual**.

1. On the **Management** page, under **Monitoring**, select **Enable with custom storage account (recommended)**.

1. Accept the defaults and on the the **Tags** blade, click **Create**.

    | Name | Value |
    | --- | --- |
    | **project** | **highexpertstore** |
    | **company** | **highexpert** |
     | | |

1. When you are done, select **Review + create**. 

1. After it passes validation, select **Create** to deploy the scale set.

    > **Note**: You might need to wait a few minutes.

1. Sign in to the [**Azure portal**](http://portal.azure.com) and open Azure Cloud Shell.

1. In Cloud Shell, start the code editor and execute command **code add-customextension-vmss.ps1**.

1. Add the following text to the script file:

  ```powershell
# Set Script 
$customConfig = @{ 
"fileUris" = (,"https://raw.githubusercontent.com/highexpert-tecnologia/highexpertstore-vmss/refs/heads/main/install-web-server-iis-highexpert-store.ps1"); 
"commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File install-web-server-iis-highexpert-store.ps1" 
} 

# Set VMSS variables
$rgname = "RG-HighExpert-Store-VMSS"
$vmssname = "vmsshestore"

# Get VMSS object 
$vmss = Get-AzVmss -ResourceGroupName $rgname -VMScaleSetName $vmssname

# Add VMSS extension 
$vmss = Add-AzVmssExtension -Name "CustomScript" -VirtualMachineScaleSet $vmss -Publisher "Microsoft.Compute" -Type "CustomScriptExtension" -TypeHandlerVersion "1.9" -Setting $customConfig

# Update VMSS 
Update-AzVmss -ResourceGroupName $rgname -Name $vmssname -VirtualMachineScaleSet $vmss
   ```

1. Press Ctrl+S to save the file. Then press Ctrl+Q to close the code editor.

1. Run the following command **./add-customextension-vmss.ps1**.

1. In **VMSSWEB**, select **Extensions** and check a new extension.

1. Select **Instances** in **vmsshestore**, click **Upgrade** for all instances.

    > **Note**: You might need to wait a few minutes.

4.  On the **Network security groups** blade, select **NSG-HighExpert-Store1_snet-backend**. 

5.  On the **NSG-HighExpert-Store1_snet-backend** blade, select **Inbound security rules** under **Settings** on the left and select **Add**.

6.  -  Source: **IP Address**

    -  Source IP addresses/CIDR ranges: **10.1.100.0/22**

    -  Source port ranges: **(*)**

    -  Destination: **IP Address**

    -  Destination IP addresses/CIDR ranges: **10.2.3.0/24**

    -  Destination port ranges: **1433**

    -  Protocol: **TCP**

    -  Action: **Allow**

    -  Priority: **302**

    -  Name: **AllowVMSSTierInboundTCP1433**

1. Test the Public IP address in Browser.

1. In the Azure portal, navigate to the **Virtual Machine Scale Sets** entry, and on the **vmsshestore** blade, select **Scaling**. 

1. On the **Scaling** blade, select the **Custom autoscale** option.

1. In the **Custom autoscale** section, specify the following settings (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Scaling mode | **Scale based on a metric** |
    | Instance limits Minimum | **1** |
    | Instance limits Maximum | **3** |
    | Instance limits Default | **1** |

1. Select **+ Add a rule**.

1. On the **Scale rule** blade, specify the following settings and select **Add** (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Time aggregation | **Maximum** |
    | Metric namespace | **Virtual Machine Host** |
    | Metric name | **Percentage CPU** |
    | VMName Operator | **=** |
    | Operator | **Greater than** |
    | Metric threshold to trigger scale action | **5** |
    | Duration (in minutes) | **5** |
    | Time grain statistics | **Maximum** |
    | Operation | **Increase count by** |
    | Instance count | **1** |
    | Cool down (minutes) | **5** |

1. Back on the **Scaling** blade, select **+ Add a rule**.

1. On the **Scale rule** blade, specify the following settings and select **Add** (leave others with their default values):

    | Setting | Value | 
    | --- | --- |
    | Time aggregation | **Average** |
    | Metric namespace | **Virtual Machine Host** |
    | Metric name | **Percentage CPU** |
    | VMName Operator | **=** |
    | Operator | **Less than** |
    | Metric threshold to trigger scale action | **1** |
    | Duration (in minutes) | **10** |
    | Time grain statistics | **Minimum** |
    | Operation | **Decrease count by** |
    | Instance count | **1** |
    | Cool down (minutes) | **5** |

1. Back on the **Scaling** blade, select **Save**.

1. In the Azure portal, start a new **Bash** session in the Cloud Shell pane. 

1. From the Cloud Shell pane, run the following to trigger autoscaling of the Azure VM Scale Set instances in the backend pool of the Azure Application Gateway (replace the `<lb_IP_address>` placeholder with the IP address of the front end of the gateway you identified earlier):

   ```Bash
   for (( ; ; )); do curl -s <lb_IP_address>?[1-1000]; done
   ```
1. In the Azure portal, on the **vmsshestore** blade, review the **CPU (average)** chart and verify that the CPU utilization of the Application Gateway increased sufficiently to trigger scaling out.

    > **Note**: You might need to wait a few minutes.

## After the Hands-on lab 

1. Delete all Azure resources created in support of this Hands-on lab.

1. End of Project Hands-on.

1. Continue in the **Mentoria Arquiteto Azure**.
