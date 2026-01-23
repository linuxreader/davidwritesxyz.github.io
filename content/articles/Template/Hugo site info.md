---
draft: true
---

Current working hugo version 
```bash
cd ~/Downloads
curl -LO https://github.com/gohugoio/hugo/releases/download/v0.152.2/hugo_extended_0.152.2_Linux-64bit.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0   0     0   0     0     0     0  --:--:-- --:--:-- --:--:--     0
100 18798k 100 18798k   0     0 16037k     0   0:00:01  0:00:01 --:--:-- 35476k

~/Downloads 
❯ tar -tzf hugo_extended_0.152.2_Linux-64bit.tar.gz
hugo
README.md
LICENSE

~/Downloads 
❯ mkdir -p ~/.local/bin
tar -xzf hugo_extended_0.152.2_Linux-64bit.tar.gz
mv hugo ~/.local/bin/
chmod +x ~/.local/bin/hugo

~/Downloads 
❯ echo $PATH | grep -q "$HOME/.local/bin" || \
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

~/Downloads 
❯ source ~/.bashrc

~/Downloads 
❯ hugo version
hugo v0.152.2-6abdacad3f3fe944ea42177844469139e81feda6+extended linux/amd64 BuildDate=2025-10-24T15:31:49Z VendorInfo=gohugoio
```

New version
```
cd ~/Downloads
curl -LO https://github.com/gohugoio/hugo/releases/download/v0.154.5/hugo_extended_0.154.5_Linux-64bit.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0   0     0   0     0     0     0  --:--:-- --:--:-- --:--:--     0
100 18798k 100 18798k   0     0 16037k     0   0:00:01  0:00:01 --:--:-- 35476k

~/Downloads 
❯ tar -tzf hugo_extended_0.154.5_Linux-64bit.tar.gz
hugo
README.md
LICENSE

~/Downloads 
❯ mkdir -p ~/.local/bin
tar -xzf hugo_extended_0.154.5_Linux-64bit.tar.gz
mv hugo ~/.local/bin/
chmod +x ~/.local/bin/hugo

~/Downloads 
❯ echo $PATH | grep -q "$HOME/.local/bin" || \
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc

~/Downloads 
❯ source ~/.bashrc

~/Downloads 
❯ hugo version
hugo v0.152.2-6abdacad3f3fe944ea42177844469139e81feda6+extended linux/amd64 BuildDate=2025-10-24T15:31:49Z VendorInfo=gohugoio
```