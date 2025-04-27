# wireguard-cheat-sheet

# Down

## Down all
```shell
if [ -n "$(sudo wg show interfaces)" ]; then sudo wg show interfaces | xargs -n1 sudo wg-quick down; else echo "No WireGuard interfaces up, chill mal 😎"; fi
```




# Connect

```shell
sudo wg-quick up main_os-CH-AT-1
```
