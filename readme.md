https://github.com/paulmartins/hugo-digital-garden-theme

To run a server:
```shell
hugo server
```

To build locally to [./public](./public):
```shell
hugo
```

To update the hugo-digital-garden-theme:
```shell
git submodule update --remote --rebase
``` 

If the commit hash for ./themes/digital-garden is not updating:
```
cd themes/digital-garden
git pull
cd - 
git submodule update --init --recursive --remote
```

[A hidden git submodule config points to a remote](.git/modules/themes/digital-garden/config). 
I pointed it to [my digital garden fork](https://github.com/james-s-w-clark/hugo-digital-garden-theme).
```shell
git submodule sync --recursive
```
The `digital-garden` folder will change hash - can see changelog by comparing hashes against https://github.com/paulmartins/hugo-digital-garden-theme/commits/main.

