# smart-internet-technologies-coursework
Project files for adaptive bitrate streaming using MPEG-DASH

# 1. FFmpeg Installation (Server - Linux Mint)
sudo apt update
sudo apt install ffmpeg

# 2. Video Transcoding with FFmpeg
Both HD videos were transcoded at 1.5Mbps, 2.0Mbps, 4.0Mbps

### Waves_on_the_beach.mp4
  
  1.5Mbps bitrate
  
    ffmpeg -i waves_on_the_beach.mp4 -b:v 1500k WavesOutput_1.5mbps.mp4
2.0Mbps bitrate
    
    ffmpeg -i waves_on_the_beach.mp4 -b:v 2000k WavesOutput_2mbps.mp4

4.0Mbps bitrate
    
    ffmpeg -i waves_on_the_beach.mp4 -b:v 4000k WavesOutput_4mbps.mp4

### Galaxy.mp4

  1.5Mbps bitrate
  
    ffmpeg -i galaxy.mp4 -b:v 1500k Galaxy_1.5Mbps.mp4
2.0Mbps bitrate
    
    ffmpeg -i galaxy.mp4 -b:v 2000k Galaxy_2Mbps.mp4


4.0Mbps bitrate
    
    ffmpeg -i galaxy.mp4 -b:v 1500k Galaxy_4Mbps.mp4

# 3. DASH Manifest Generation

    ffmpeg -i galaxy.mp4 \
      -map 0 -b:v 1500k -b:v 2000k -b:v 4000k \
      -use_template 1 -use_timeline 1 \
      -window_size 5 -extra_window_size 5 \
      -seg_duration 4 \
      -f dash manifest.mpd

# 4. Apache2 Hosting (Server)

### Install Apache2
    sudo apt install apache2#


### Applying permissions on web root
    cd /var/www/html
    
    sudo chmod -R 755 /var/www/html

# 5. DASH Playback (Client)

<img width="1622" height="642" alt="image" src="https://github.com/user-attachments/assets/d906a92e-7a6d-427f-9f88-a0c1a52fff00" />

Screenshot showing successful DASH Playbacck using the DASH -IF reference player. Manifest loaded from http://192.168.1.182/manifest.mpd at 1500kbps

# 6. IPERF Bandwidth Testing

Start IPERF server:

    iperf3 -s

Connect from client with limited bandwidth:

    iperf3 -c 192.168.1.182 -b 1M

<img width="940" height="587" alt="image" src="https://github.com/user-attachments/assets/f6286634-3142-4c38-8ef8-170cfd03e5ae" />


This simulates congestion and forces DASH to adapt to lower bitrates.

# 7. Traffic Control (TC) Artefacts

### TBF

    sudo tc qdisc add dev enp0s3 root tbf rate 2.5mbit burst 20kb latency 50ms

### HTB

    sudo tc qdisc add dev enp0s3 root handle 1: htb default 12
    sudo tc class add dev enp0s3 parent 1: classid 1:1 htb rate 2.5mbit ceil 5mbit burst 20kb

### Ingress Policing

    sudo tc qdisc add dev enp0s3 handle ffff: ingress
    sudo tc filter add dev enp0s3 parent ffff: protocol ip prio 1 police rate 3.5mbit burst 10kb drop flowid :1

### Remove artefacts

    sudo tc qdisc del dev enp0s3 root
    sudo tc qdisc del dev enp0s3 ingress
<img width="797" height="97" alt="image" src="https://github.com/user-attachments/assets/750380bf-1c25-4357-963e-1e5656d5eccf" />


