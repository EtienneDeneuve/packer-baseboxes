# Packer

## Documentation officielle

La documentation est ici :
[packer doc](https://www.packer.io/docs)

## Installation des pré-requis pour Packer

```powershell
Write-Output "Installation of Chocolatey"
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1’))
$packages = ('VisualStudioCode', 'git', 'terraform', 'packer')
choco upgrade $packages -y
```

## Fonctionnement de Packer

Packer repose sur un fichier Json au format suivant:

```json
{
  "variables": {},
  "builders": [{}],
  "provisioners": [{}]
}
```

## Préparation du workspace

```powershell
mkdir $($env:USERPROFILE + "\Documents\git")
cd  $($env:USERPROFILE + "\Documents\git")
git clone https://github.com/taliesins/packer-baseboxes.git
cd packer-baseboxes
```

## Builder une image

### Azure

#### Préparation

Pour builder une image pour azure vous avez besoin d'un abonnement azure actif.
Il faudra créer un service principal et récupérer l'application id ainsi que l'application secret. Vous devez également récupérer l'id de votre tenant et de votre subscription.

Voici des liens pour les 3 manières :

- Use portal to create an Azure Active Directory application and service principal that can access resources : [lien](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)
- Create an Azure service principal with Azure PowerShell : [lien](https://docs.microsoft.com/en-us/powershell/azure/create-azure-service-principal-azureps?view=azurermps-4.2.0)
- Create an Azure service principal with Azure CLI 2.0 : [lien](https://docs.microsoft.com/en-us/cli/azure/create-an-azure-service-principal-azure-cli?toc=%2fazure%2fazure-resource-manager%2ftoc.json)

#### Création du template

1. Créer un fichier azure-ubuntu-18.04.json :

2. Ajoutez le contenu de base :

```json
{
  "variables": {},
  "builders": [{}],
  "provisioners": [{}]
}
```

3. Configurez le template tel que (en remplaçant les valeurs par celle récupéré de l'étape 1):

```json
{
  "variables": {},
  "builders": [
    {
      "type": "azure-arm",

      "client_id": "<Change me>",
      "client_secret": "<Change me>",
      "subscription_id": "<Change me>",
      "tenant_id": "<Change me>",

      "managed_image_name": "ubuntu-18.04-latest",
      "managed_image_resource_group_name": "rg-packer-images",

      "os_type": "Linux",
      "image_publisher": "Canonical",
      "image_offer": "UbuntuServer",
      "image_sku": "18.04-LTS",

      "azure_tags": {
        "dept": "engineering"
      },

      "location": "West Europe",
      "vm_size": "Standard_A2"
    }
  ],
  "provisioners": [
    {
      "execute_command": "chmod +x {{ .Path }}; {{ .Vars }} sudo -E sh '{{ .Path }}'",
      "inline": [
        "DEBIAN_FRONTEND=noninteractive apt-get -y update"
        "DEBIAN_FRONTEND=noninteractive apt-get -y upgrade"
      ],
      "inline_shebang": "/bin/sh -x",
      "type": "shell"
    },
    {
      "execute_command": "chmod +x {{ .Path }}; {{ .Vars }} sudo -E sh '{{ .Path }}'",
      "inline": [
        "/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync"
      ],
      "inline_shebang": "/bin/sh -x",
      "type": "shell"
    }
  ]
}
```

4. Créer un resources groupe "rg-packer-images"

```bash
az login
az group create -l westeurope -n rg-packer-images
```

5. Lancez la build

```bash
packer build .\azure-ubuntu-18.04.json
```

### Hyper-v sur windows 10

> L'antivirus Dell nous empêche de faire fonctionner packer directement sur nos postes de travail.  
> Pour l'utiliser quand même, faites une Vm Windows Server ou Windows 10 avec la nested virtualisation activée :  
> [Run Hyper-V in VM](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/nested-virtualization)

#### Ubuntu 18.04

```powershell
PS C:\Users\etienne_deneuve\Documents\git\packer-baseboxes> packer build .\hyperv-ubuntu-18.04.json
```

#### Windows Server 2012 R2

```powershell
PS C:\Users\etienne_deneuve\Documents\git\packer-baseboxes> packer build .\hyperv-windows-2016-serverstandard-amd64.json
```

### VMWare (Workstation, Fusion)

#### Windows Server 2012 R2
```powershell
PS C:\Users\etienne_deneuve\Documents\git\packer-baseboxes> packer build .\vmware-windows-2016-serverstandard-amd64.json
```