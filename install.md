From github [ErikOostveen/SuperColliderOnUniHiker](https://github.com/ErikOostveen/SuperColliderOnUniHiker)

```
sudo apt remove '*jack*'
sudo apt-get update
sudo apt-get upgrade
# This may take a while ...
sudo apt-get dist-upgrade
cd ~ 
git clone https://github.com/jackaudio/jack2.git 
cd jack2 
./waf configure --alsa --prefix=/usr --libdir=/usr/lib/x86_64-linux-gnu 
./waf 
su -c "./waf install" 
sudo ldconfig 
sudo apt-get install jackd # (Select the onscreen "Yes")
sudo apt-get install libjack-jackd2-dev 
```
```
sudo apt-get install libsamplerate0-dev libsndfile1-dev libasound2-dev libavahi-client-dev libreadline-dev libfftw3-dev libudev-dev libncurses5-dev cmake git

sudo sh -c "echo @audio - memlock 256000 >> /etc/security/limits.conf"
sudo sh -c "echo @audio - rtprio 75 >> /etc/security/limits.conf"

sudo reboot now
```
```
sudo cmake --build . --config Release --target install
sudo ldconfig
```

```
aplay -l # check audio interface ID
echo /usr/bin/jackd -r -d alsa -d hw:1 > ~/.jackdrc
```

Test demofile:
```
# cat demoSound.scd
s.waitForBoot{
 (
  play{a=ar(PinkNoise,5e-4);ar(GVerb,({ar(Ringz,a*LFNoise1.kr(0.2),exprand(60,8000),3)}!40).sum,50,99).tanh}
 )
}
```

Add jack env:
```
export JACK_NO_AUDIO_RESERVATION=1 # EDIT

```

So autostart script
```
$ cat autostart.sh
#!/bin/bash
export PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/local/games:/usr/games
export DISPLAY=:0.0
export JACK_NO_AUDIO_RESERVATION=1 # EDIT

# Retrieve .scd patches
scdfile=(patches/*.scd)
#echo --------------------------------------------------------------------
#echo ARRAY INPUT
#echo ${scdfile[@]}

previousSelectedFile=$(<selectedfile.txt)
#echo --------------------------------------------------------------------
#echo PATCH TO BE REMOVED FROM ARRAY INPUT
#echo $previousSelectedFile

for target in "${previousSelectedFile[@]}"; do
  for i in "${!scdfile[@]}"; do
    if [[ ${scdfile[i]} = $target ]]; then
      unset 'scdfile[i]'
    fi
  done
done

for i in "${!scdfile[@]}"; do
    new_scdfile+=( "${scdfile[i]}" )
done
scdfile=("${new_scdfile[@]}")
unset new_scdfile
scdfile=( $(shuf -e "${scdfile[@]}") )

#echo --------------------------------------------------------------------
#echo NEW ARRAY INPUT
#echo ${scdfile[@]}

rand=$[$RANDOM % ${#scdfile[@]}]
selectedfile=${scdfile[$rand]}
#echo --------------------------------------------------------------------
#echo PATCH SELECTED
#echo $selectedfile

# Override selected patch (do the same in reload.sh!)
# -------------------------------------------------------
#selectedfile="patches/4_PD_new_sing_saw.scd"
# -------------------------------------------------------

echo $selectedfile > selectedfile.txt
#echo "Selected patch"
#echo $selectedfile
source .virtualenvs/PlanetDrone/bin/activate
python3 ~/.virtualenvs/PlanetDrone/PlanetDrone.py &
sleep 5
# Start Super Collider and run selected patch
sclang ~/$selectedfile
```