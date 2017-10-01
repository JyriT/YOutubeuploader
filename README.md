Raspista livevideota Youtubeen:
(Ylläpito tehnee tästä tiedoston jos sen hyväksi katsoo) 
Oletan että lukija on perillä jotenkin linuksin sielunelosta ja hanskaa perusjutskat. 
Pöydälläsi nököttää Raspberry Pi 3 tai ZeroW jossa on raspin oma kamera liitettynä.
Olet saanut otettua ensimmäiset kuvat kameralla käyttäen raspistill komentoa.
Jotta kameran sensorin tuottama bittivirta saadaan survottua youtubeen, tarvitaan pari välipalikkaa:
Raspivid ja FfMpeg. 
Raspivid löytyy samasta paketista kuin raspistill, ffmepg joka tulee raspbianin mukana ei toimi. Se pitää kääntää sorsasta. 
Ffmpeg:
SSH:lla kojeeseen sisään ja sitten loihditaan: 
git clone git://source.ffmpeg.org/ffmpeg.git
cd ffmpeg
./configure --enable-gpl --enable-nonfree --enable-libx264 --enable-libmp3lame
sudo make -j$(nproc) && make install
Jos kaikki meni jeppis, onnittele itseäsi, jos ei, raspistasi puuttuu luultavasti kehityskirjastoja, virhekoodit antavat vinkkejä mitä pitää asentaa.
Vahva ehdokas puuttujista on x264. 
Asennus tälläi:
git clone git://git.videolan.org/x264
cd x264
./configure --host=arm-unknown-linux-gnueabi --enable-static --disable-opencl
make
sudo make install
Jos muuta uupuu, Asenna puuttuvat ja aja puskaan mennyt steppi uudellen.
Jotta kuva saadaan kulkemaan interrnettiin, tarjoillaan koneelle esim cronissa @reboot ajastuksella, tai init.rc:ssä seuraava skripti.
Striimiavain löytyy Youtube creatorin alta, live streaming kohdasta, ihan alhaalta.
#!/bin/bash
wait 20
RTMP_URL=rtmp://a.rtmp.youtube.com/live2
STREAM_KEY=<tähän striimiavain joka on encoder setupissa>
while :
do
raspivid -n -vf -hf -md 5 -t 0 -fps 25 -b 1000000 -o - | ffmpeg -ar 8000 -ac 2 -acodec pcm_s16le -f s16le -ac 2 -i /dev/zero -f h264 \
-i - -vcodec copy -acodec aac -ab 64k -g 50 -strict experimental -f flv $RTMP_URL/$STREAM_KEY
sleep 2
done
Eikun katselemaan.
