# Slavearmy part 2

Koska julkaisin aikaisemmat skriptini, joilla loin orjia useammalle koneelle, niin uskon muiden hyödyntävän niitä ja aijon nyt parannella niitä paremman tuloksen saamiseksi.

Aloitin luomalla uuden virtualbox laatikon, johon asensin valmiiksi puppetin ja asetin kaikki tiedot masterpalvelinta varten. Näin säästän aikaa virtuaalikoneiden luonti vaiheessa, kun virtuaali koneiden ei tarvitse hakea netistä ja asentaa kaikkia tiedostoja kuten aikaisemmin.
Laatikon luominen onnistui, kun pyöräytin yhden koneen käyntiin johon ajettiin provisio skriptit ja tämän jälkeen kun laatikko oli ylhäällä käytin komentoa:
Pohjana custom laatikossani oli envimation/ubuntu-xenial.
```
vagrant package --output custom.box
```
Joka sulki käynnissä olevan laatikon ja loi siitä uuden base boxin, jolla voi jatkossa asentaa useampia koneita.
Latasin kyseisen laatikon omalle palvelimelleni ja vaihdoin Vagrantfileen kohtan config.vm.box osoitteen, josta uusi lattikko löytyy.
Koska provisioskripti on jo laatikkoon asennettuna, voin poistaa Vagrantfilestä provisioskriptin asennus kohdan.

Uusi Vagrantfile näyttää siis tältä:
```
Vagrant.configure(2) do |config|
  config.vm.box = "http://timonen.me/~tommi/boxit/custom.box"
  config.vm.usable_port_range = (2200..20000)

  r = rand(36**10).to_s(36)

  (1..200).each do |i|
    config.vm.define "slave#{i}" do |slave|
	slave.vm.hostname = "Slave#{i}-#{r}"

    slave.vm.provider "virtualbox" do |vb|
       vb.memory = 150
       vb.linked_clone = true
    end
    end
  end
end
```

Tämän jälkeen aloin miettimään miten saisin mahdutettua enemmän virtuaalikoneita yhteen koneeseen. Rajoituksenahan koulun koneissa on ollut se, että ajan linuxia, joka luo virtuaalikoneet, aina RAM muistissa. RAM muisti siis rajaa kaiken toiminnan.
Yhtäkkiä mieleeni juolahti, että voisin mountata aina koulun koneen kovalevyn ja asentaa virtuaalikoneiden .vdi virtuaaliset kovalevyt sinne, jolloin tilaa säästyy RAM:ilta ja mahtuu taas pari virtuaalista konetta enemmän.
Tämän jälkeen tajusin, että jos kerran mounttaan kovalevyn, niin voin hyödyntää myös swap tiedostoa, kuten käytin omalle koneelleni slaveja asentaessa.
Loin siis bash skriptin, joka toimii kuten edellisessä yrityksessä käytetty skripti, mutta lisäksi se hoitaa kaikki nämä vaiheet puolestani.
Lisäsin skriptiin myös vaiheen, jossa se asentaa uusimman vagrant version oman palvelimeni kautta, sillä xubuntun omien repositoryjen kautta saatu vagrant on hieman vanhentunut.

Uusi starter skripti näyttää siis tältä:
```
#!/bin/bash
GREEN='\033[1;32m'
NC='\033[0m'
sudo cp Vagrantfile /home/.
echo "\n ${GREEN}vagrantfile copied! ${NC} \n \n"
cd /home/
sudo wget http://timonen.me/~tommi/boxit/vagrant.deb
printf "\n ${GREEN} Vagrant downloaded! ${NC} \n \n"
sudo dpkg -i vagrant.deb
printf "\n ${GREEN} Vagrant installed! ${NC} \n \n"
sudo apt-get update
sudo apt-get -y install virtualbox
printf "\n ${GREEN} Virtualbox installed! ${NC} \n \n"
# fdisk -l to check disks
sudo mount -t ext4 /dev/sda1 /mnt
printf "\n ${GREEN} HDD mounted! ${NC} \n \n \n \n"
sudo vboxmanage setproperty machinefolder /mnt
printf "\n ${GREEN} Machine folder changed! ${NC} \n \n"
sudo dd if=/dev/zero of=/mnt/swapfile1 bs=1024 count=10240000
printf "\n ${GREEN} Swap file created! ${NC} \n \n"
sudo chown root:root /mnt/swapfile1
sudo chmod 0600 /mnt/swapfile1
sudo mkswap /mnt/swapfile1
sudo swapon /mnt/swapfile1
printf "\n ${GREEN} Swap file applied! ${NC} \n \n"
sudo sysctl vm.swappiness=100
printf "\n ${GREEN} Swappiness enabled and starting vagrant! ${NC} \n \n"
sudo vagrant up
```


