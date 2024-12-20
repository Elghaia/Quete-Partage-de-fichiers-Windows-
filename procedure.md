# Prérequis
Un serveur Windows Server 2022 avec le rôle ADDS déployé 

Un poste client Windows 10 

Des groupes AD créés : "RH", "Comptabilité", "Direction"

Des utilisateurs appartenant à ces groupes

Le rôle Serveur de fichiers installé sur le serveur


# Creation d'un Script PowerShell partage.PS1

  ```powershell
# Nom du serveur et du partage
$serveur = "SRVWIN01"
$partage = "Docs"
$dossier_partage = "C:\Documents_Entreprise"

# Création du dossier et des sous-dossiers si nécessaire
New-Item -Path $dossier_partage -ItemType directory -Force
New-Item -Path "$dossier_partage\RH" -ItemType directory -Force
New-Item -Path "$dossier_partage\Comptabilité" -ItemType directory -Force
New-Item -Path "$dossier_partage\Direction" -ItemType directory -Force

# Création du partage SMB
New-SmbShare -Name $partage -Path $dossier_partage

# Configuration des permissions NTFS
$groupe_rh = "wilder.lan\RH"
$groupe_compta = "wider.lan\Comptabilité"
$groupe_direction = "wilder.lan\Direction"
function Add-AccessRule {
    param(
        [Parameter(Mandatory)]
        [string]$path,
        [string]$identityReference,
        [string]$rights
    )

    $acl = Get-Acl $path
    $rule = New-Object System.Security.AccessControl.FileSystemAccessRule($identityReference, $rights, "Allow")
    $acl.AddAccessRule($rule)
    Set-Acl -Path $path -AclObject $acl
}
# Ajouter les permissions aux groupes
Add-AccessRule -path "C:\Documents_Entreprise\RH" -identityReference "wilder.lan\RH" -rights "FullControl"
Add-AccessRule -path "C:\Documents_Entreprise\Comptabilité" -identityReference "wilder.lan\Comptabilité" -rights "FullControl"
Add-AccessRule -path "C:\Documents_Entreprise\Direction" -identityReference "wilder.lan\Direction" -rights "FullControl"

# Donner la permission sur tout les dossiers au groupe direction 
Add-AccessRule -path "C:\Documents_Entreprise\RH" -identityReference "wilder.lan\Direction" -rights "FullControl"
Add-AccessRule -path "C:\Documents_Entreprise\Comptabilité" -identityReference "wilder.lan\Direction" -rights "FullControl"

# Donner la permission en lecture seul aux utilisateurs du domaine

Add-AccessRule -path "C:\Documents_Entreprise" -identityReference "Everyone" -rights "Read"

# Vérifier les partages
Get-SmbShare
  ```
# Execution du script

.\partage.ps1

# Configuration du lecteur réseau sur le client
New-PSDrive -Name "Z" -PSProvider FileSystem -Root "\\SRVWIN01\Docs" -Persist

# Tests des accès
Get-ChildItem -Path "Z:\"
