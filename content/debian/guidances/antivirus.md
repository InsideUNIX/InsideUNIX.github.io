```
clamav clamav-daemon clamav-base clamav-freshclam libclamav9 libmspack0 libtfm1  clamdscan
clamtk
```

```
sudo systemctl enable clamav-daemon
sudo systemctl enable clamav-freshclam
sudo systemctl start clamav-daemon
sudo systemctl start clamav-freshclam
```