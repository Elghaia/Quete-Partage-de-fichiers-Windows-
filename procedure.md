## Prérequis
Un serveur Windows Server 2022 avec le rôle ADDS déployé
Un poste client Windows 10
Des groupes AD créés : "RH", "Comptabilité", "Direction"
Des utilisateurs appartenant à ces groupes


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

# Attribuer les permissions NTFS
Set-Acl -Path "$dossier_partage\RH" -AccessRule "IdentityReference=$groupe_rh;AccessControlType='Allow';Rights='FullControl'"
Set-Acl -Path "$dossier_partage\Comptabilité" -AccessRule "IdentityReference=$groupe_compta;AccessControlType='Allow';Rights='FullControl'"
Set-Acl -Path $dossier_partage -AccessRule "IdentityReference=$groupe_direction;AccessControlType='Allow';Rights='FullControl'"
Set-Acl -Path $dossier_partage -AccessRule "IdentityReference='Everyone';AccessControlType='Allow';Rights='Read'"

# Vérifier les partages
Get-SmbShare
  ```
# Execution du script

.\partage.ps1

# Configuration du lecteur réseau sur le client
New-PSDrive -Name "Z" -PSProvider FileSystem -Root "\\SRVWIN01\Docs" -Persist

# Tests des accès
Get-ChildItem -Path "Z:\"
