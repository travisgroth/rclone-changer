````
# bacula-dir.conf
# pool
Pool {
  Name = Pool_Pcloud
  Pool Type = Backup
  Recycle = yes
  AutoPrune = yes
  Storage = Storage_Pcloud
  Maximum Volume Bytes = 1073741824 # recomendado
  AutoPrune = yes
  Volume Retention = 4 weeks
}

# storage
Autochanger {
  Name = Storage_Pcloud
  Address = 192.168.1.87
  SDPort = 9103
  Password = "6Nv2Tt2CJAf1TZA6t1cHtktA5aBUAARmJb/4BSckBLRm"
  Device = Device-Pcloud
  Media Type = pcloud
  Autochanger = Storage_Pcloud
  Maximum Concurrent Jobs = 10
}


# bacula-sd.conf

Autochanger {
  Name = "Autochanger_Pcloud"
  Device = Device-Pcloud
  Changer Device = "pcloud:bacula_backup"
  Changer Command = "/usr/local/bin/rclone-changer %c %o %S %a"
}

Device {
  Name = "Device-Pcloud"
  Media Type = pcloud
  Maximum Changer Wait = 18000 # pode ser necessário mudar de acordo com o tamanho dos volumes
  Archive Device = /mnt/vtapes/tape # precisa corresponder ao arquivo criado no passo anterior
  Autochanger = yes
  LabelMedia = yes
  Random Access = Yes
  AutomaticMount = no
  RemovableMedia = no
  AlwaysOpen = no
  Spool Directory = /mnt/bacula-spool
  Maximum Spool Size = 524288000
}
````
